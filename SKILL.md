---
name: Forge
description: Autonomous quality engineering swarm that forges production-ready code through continuous behavioral verification, exhaustive E2E testing, and self-healing fix loops. Combines DDD+ADR+TDD methodology with BDD/Gherkin specifications, 7 quality gates, defect prediction, chaos testing, and cross-context dependency awareness. Architecture-agnostic — works with monoliths, microservices, modular monoliths, and any bounded-context topology.
---

# Forge — Autonomous Quality Engineering Swarm

**Quality forged in, not bolted on.**

Forge is a self-learning, autonomous quality engineering swarm that unifies three approaches into one:

| Pillar | Source | What It Does |
|--------|--------|--------------|
| **Build** | DDD+ADR+TDD methodology | Structured development with quality gates, defect prediction, confidence-tiered fixes |
| **Verify** | BDD/Gherkin behavioral specs | Continuous behavioral verification — the PRODUCT works, not just the CODE |
| **Heal** | Autonomous E2E fix loop | Test → Analyze → Fix → Commit → Learn → Repeat |

**"DONE DONE"** means: the code compiles AND the product behaves as specified. Every Gherkin scenario passes. Every quality gate clears. Every dependency graph is satisfied.

---

## ARCHITECTURE ADAPTABILITY

Forge adapts to any project architecture. Before first run, it discovers your project structure:

### Supported Architectures

| Architecture | How Forge Adapts |
|-------------|-----------------|
| **Monolith** | Single backend process, all contexts in one codebase. Forge runs all tests against one server. |
| **Modular Monolith** | Single deployment with bounded contexts as modules. Forge discovers modules and tests each context independently. |
| **Microservices** | Multiple services. Forge discovers service endpoints, tests each service, validates inter-service contracts. |
| **Monorepo** | Multiple apps/packages in one repo. Forge detects workspace structure (Turborepo, Nx, Lerna, Melos, Cargo workspace). |
| **Mobile + Backend** | Frontend app with backend API. Forge starts backend, then runs E2E tests against it. |
| **Full-Stack Monolith** | Frontend and backend in same deployment. Forge tests through the UI layer against real backend. |

### Project Discovery

On first invocation, Forge analyzes the project to build a context map:

```bash
# Forge automatically discovers:
# 1. Backend technology (Rust/Cargo, Node/npm, Python/pip, Go, Java/Maven/Gradle, .NET)
# 2. Frontend technology (Flutter, React, Next.js, Vue, Angular, SwiftUI, Kotlin/Compose)
# 3. Test framework (integration_test, Jest, Pytest, Go test, JUnit, xUnit)
# 4. Project structure (monorepo layout, service boundaries, module boundaries)
# 5. API protocol (REST, GraphQL, gRPC, WebSocket)
# 6. Build system (Make, npm scripts, Gradle tasks, Cargo features)
```

Forge stores the discovered project map:

```json
{
  "architecture": "mobile-backend",
  "backend": {
    "technology": "rust",
    "buildCommand": "cargo build --release --features test-endpoints",
    "runCommand": "cargo run --release --features test-endpoints",
    "healthEndpoint": "/health",
    "port": 8080,
    "migrationCommand": "cargo sqlx migrate run"
  },
  "frontend": {
    "technology": "flutter",
    "testCommand": "flutter drive --driver=test_driver/integration_test.dart --target={target}",
    "testDir": "integration_test/e2e/",
    "specDir": "integration_test/e2e/specs/"
  },
  "contexts": ["identity", "rides", "payments", "..."],
  "testDataSeeding": {
    "method": "api",
    "endpoint": "/api/v1/test/seed",
    "authHeader": "X-Test-Key"
  }
}
```

### Configuration Override

Projects can provide a `forge.config.yaml` at the repo root to override auto-discovery:

```yaml
# forge.config.yaml (optional — Forge auto-discovers if absent)
architecture: microservices
backend:
  services:
    - name: auth-service
      port: 8081
      healthEndpoint: /health
      buildCommand: npm run build
      runCommand: npm start
    - name: payment-service
      port: 8082
      healthEndpoint: /health
      buildCommand: npm run build
      runCommand: npm start
frontend:
  technology: react
  testCommand: npx cypress run --spec {target}
  testDir: cypress/e2e/
  specDir: cypress/e2e/specs/
contexts:
  - name: identity
    testFile: auth.cy.ts
    specFile: identity.feature
  - name: payments
    testFile: payments.cy.ts
    specFile: payments.feature
dependencies:
  identity:
    blocks: [payments, orders]
  payments:
    depends_on: [identity]
    blocks: [orders]
```

---

## CRITICAL: NO MOCKING OR STUBBING ALLOWED

**ABSOLUTE RULE: This skill NEVER uses mocking or stubbing of the backend API.**

- ALL tests run against the REAL backend API
- NO mocking frameworks for API calls (no `mockito`, `wiremock`, `MockClient`, `nock`, `msw`, `httpretty`, etc.)
- NO stubbed responses or fake data from API endpoints
- The backend MUST be running and healthy before any tests execute
- Test data is seeded through REAL API calls, not mocked state

**Why No Mocking:**
- Mocks hide real integration bugs
- Mocks create false confidence
- Mocks don't test the actual data flow
- Real API tests catch serialization, validation, and timing issues

---

## PHASE 0: BACKEND SETUP (MANDATORY FIRST STEP)

**BEFORE ANY TESTING, the backend MUST be built, compiled, and running.**

This is the FIRST thing the skill does — no exceptions.

### Step 1: Check and Start Backend

```bash
# 1. Read project config or auto-discover backend settings
# 2. Check if backend is already running
curl -s http://localhost:${BACKEND_PORT}/${HEALTH_ENDPOINT} || {
  echo "Backend not running. Starting..."

  # 3. Navigate to backend directory
  cd ${BACKEND_DIR}

  # 4. Ensure environment is configured
  cp .env.example .env 2>/dev/null || true

  # 5. Build the backend
  ${BUILD_COMMAND}

  # 6. Run database migrations (if applicable)
  ${MIGRATION_COMMAND}

  # 7. Start backend (background)
  nohup ${RUN_COMMAND} > backend.log 2>&1 &
  echo $! > backend.pid

  # 8. Wait for backend to be healthy (up to 60 seconds)
  for i in {1..60}; do
    if curl -s http://localhost:${BACKEND_PORT}/${HEALTH_ENDPOINT} | grep -q "ok\|healthy\|UP"; then
      echo "Backend healthy on port ${BACKEND_PORT}"
      break
    fi
    sleep 1
  done
}
```

### Step 2: Verify Backend Health

```bash
# Verify critical endpoints are responding
curl -s http://localhost:${BACKEND_PORT}/${HEALTH_ENDPOINT} | jq .

# Verify test fixtures endpoint (for seeding)
curl -s -H "${TEST_AUTH_HEADER}" http://localhost:${BACKEND_PORT}/${TEST_STATUS_ENDPOINT} | jq .
```

### Step 3: Contract Validation

```bash
# Verify API spec matches running API (if OpenAPI/Swagger available)
curl -s http://localhost:${BACKEND_PORT}/${OPENAPI_ENDPOINT} > /tmp/live-spec.json

# Store contract snapshot for regression detection
npx @claude-flow/cli@latest memory store \
  --key "contract-snapshot-$(date +%s)" \
  --value "$(cat /tmp/live-spec.json | head -c 5000)" \
  --namespace forge-contracts
```

### Step 4: Seed Test Data (Real API Calls)

```bash
# Seed test data through REAL API — adapt to your project's seeding endpoint
curl -X POST http://localhost:${BACKEND_PORT}/${SEED_ENDPOINT} \
  -H "Content-Type: application/json" \
  -H "${TEST_AUTH_HEADER}" \
  -d '${SEED_PAYLOAD}'
```

---

## PHASE 1: BEHAVIORAL SPECIFICATION

**Before testing, verify Gherkin specs exist for the target bounded context.**

Behavioral specifications define WHAT the product does from the user's perspective. Every test traces back to a Gherkin scenario. If tests pass but specs fail, the product is broken.

### Spec Location

Gherkin specs are stored alongside tests:

```
${SPEC_DIR}/
├── [context-a].feature
├── [context-b].feature
├── [context-c].feature
└── ...
```

The exact location depends on your project's test structure. Forge auto-discovers this from the project map.

### Spec-to-Test Mapping

Each Gherkin `Scenario` maps to exactly one test function. The mapping is tracked:

```gherkin
Feature: [Context Name]
  As a [user role]
  I want to [action]
  So that [outcome]

  Scenario: [Descriptive scenario name]
    Given [precondition]
    When [action]
    Then [expected result]
    And [additional verification]
```

### Missing Spec Generation

If specs are missing for a target context, the Specification Verifier agent creates them:

1. Read the screen/component/route implementation files for the context
2. Extract all user-visible features, interactions, and states
3. Generate Gherkin scenarios covering every cyclomatic path
4. Write to `${SPEC_DIR}/[context].feature`
5. Map each scenario to its corresponding test function

---

## PHASE 2: CONTRACT & DEPENDENCY VALIDATION

### Contract Validation

Before running tests, verify API response schemas match expected DTOs:

```bash
# For each endpoint the context uses:
# 1. Make a real API call
# 2. Compare response structure against expected DTO/schema
# 3. Flag any mismatches as contract violations
```

Contract violations are treated as Gate 7 failures and must be resolved before functional testing proceeds.

### Dependency Graph

Bounded contexts have dependencies. When a fix touches context X, all contexts that depend on X must be re-tested.

```yaml
# Context Dependency Map — define in forge.config.yaml or auto-discover
# Example for a typical application:
#
# authentication:
#   depends_on: []
#   blocks: [orders, payments, profile, messaging]
#
# payments:
#   depends_on: [authentication]
#   blocks: [orders, subscriptions]
#
# orders:
#   depends_on: [authentication, payments]
#   blocks: [reviews, notifications]
```

### Cascade Re-Testing

When Bug Fixer modifies a file in context X:

1. Identify which context X belongs to
2. Look up all contexts in `blocks` list for X
3. After X's tests pass, automatically re-run tests for blocked contexts
4. If a cascade failure occurs, trace it back to the original fix

---

## PHASE 3: SWARM INITIALIZATION

```bash
# Initialize anti-drift swarm for Forge
npx @claude-flow/cli@latest swarm init --topology hierarchical --max-agents 10 --strategy specialized

# Load previous fix patterns from memory
npx @claude-flow/cli@latest memory search --query "forge fix patterns" --namespace forge-patterns

# Check current coverage and gate status
npx @claude-flow/cli@latest memory retrieve --key "forge-coverage-status" --namespace forge-state

# Load confidence tiers
npx @claude-flow/cli@latest memory search --query "confidence tier" --namespace forge-patterns

# Check defect predictions for target context
npx @claude-flow/cli@latest memory search --query "defect prediction" --namespace forge-predictions
```

---

## PHASE 4: SPAWN AUTONOMOUS AGENTS

Claude Code MUST spawn these 8 agents in a SINGLE message with `run_in_background: true`:

```javascript
// Agent 1: Specification Verifier
Task({
  prompt: `You are the Specification Verifier agent. Your mission:

    1. VERIFY backend is running: curl -sf http://localhost:${BACKEND_PORT}/${HEALTH_ENDPOINT}
    2. Check if Gherkin specs exist for the target bounded context:
       - Look in the project's spec directory
    3. If specs are MISSING:
       - Read the screen/component/route implementation files for the context
       - Extract all user-visible features, interactions, states
       - Generate Gherkin feature files with scenarios for every cyclomatic path
       - Write specs to the correct location
    4. If specs EXIST:
       - Read current implementations
       - Compare against existing scenarios
       - Flag scenarios that no longer match implementation (stale specs)
       - Generate new scenarios for uncovered features
    5. Create spec-to-test mapping:
       - Each Scenario name → test function name
       - Store mapping in memory for Test Runner
    6. Store results:
       npx @claude-flow/cli@latest memory store --key "specs-[context]-[timestamp]" \
         --value "[spec status JSON]" --namespace forge-specs

    Output: List of all Gherkin scenarios with their mapped test functions, and any gaps found.`,
  subagent_type: "researcher",
  description: "Spec Verification",
  run_in_background: true
})

// Agent 2: Test Runner
Task({
  prompt: `You are the Test Runner agent. Your mission:

    1. VERIFY backend is running
    2. Check defect predictions from memory:
       npx @claude-flow/cli@latest memory search --query "defect prediction [context]" --namespace forge-predictions
       - Run predicted-to-fail tests FIRST for faster convergence
    3. Run the E2E test suite for the specified context using the project's test command
    4. Capture ALL test output including stack traces
    5. Parse failures into structured format:
       {testId, gherkinScenario, error, stackTrace, file, line, context}
    6. Map each failure to its Gherkin scenario (from spec-to-test mapping)
    7. Store results in memory for other agents:
       npx @claude-flow/cli@latest memory store \
         --key "test-run-[timestamp]" \
         --value "[parsed results JSON]" \
         --namespace forge-results`,
  subagent_type: "tester",
  description: "Test Runner",
  run_in_background: true
})

// Agent 3: Failure Analyzer
Task({
  prompt: `You are the Failure Analyzer agent. Your mission:

    1. Monitor memory for new test results from Test Runner
    2. For each failure, analyze:
       - Root cause category: element-not-found, assertion-failed, timeout,
         api-mismatch, navigation-error, state-error, contract-violation
       - Affected file and line number
       - Which Gherkin scenario is violated
       - Impact on dependent contexts (check dependency graph)
    3. Search memory for matching fix patterns with confidence tiers:
       npx @claude-flow/cli@latest memory search \
         --query "[error pattern]" --namespace forge-patterns
    4. If pattern found with confidence >= 0.85 (Gold+):
       - Recommend auto-apply
       - Include pattern key and success rate
    5. If pattern found with confidence >= 0.75 (Silver):
       - Suggest fix but flag for review
    6. If no matching pattern:
       - Perform root cause analysis from first principles
       - Generate fix hypothesis
    7. Store analysis in memory for Bug Fixer:
       npx @claude-flow/cli@latest memory store \
         --key "analysis-[testId]-[timestamp]" \
         --value "[analysis JSON]" \
         --namespace forge-results`,
  subagent_type: "researcher",
  description: "Failure Analyzer",
  run_in_background: true
})

// Agent 4: Bug Fixer
Task({
  prompt: `You are the Bug Fixer agent. Your mission:

    1. Retrieve failure analysis from memory
    2. For each failure, apply fix using confidence-tiered approach:

       PLATINUM (>= 0.95 confidence):
       - Auto-apply the stored fix pattern immediately
       - No review needed

       GOLD (>= 0.85 confidence):
       - Auto-apply the stored fix pattern
       - Flag in commit message for awareness

       SILVER (>= 0.75 confidence):
       - Read the failing test file and source file
       - Apply suggested fix with extra verification
       - Run targeted test before proceeding

       BRONZE or NO PATTERN:
       - Read the failing test file
       - Read the source file causing the failure
       - Implement fix from first principles
       - Use defensive patterns appropriate to the test framework

    3. After fixing, identify affected context:
       - Check dependency graph for cascade impacts
       - Flag dependent contexts for re-testing

    4. Store the fix pattern with initial confidence:
       npx @claude-flow/cli@latest memory store \
         --key "fix-[error-type]-[hash]" \
         --value '{"pattern":"[fix]","confidence":0.75,"tier":"silver","applied":1,"successes":0}' \
         --namespace forge-patterns

    5. Signal Test Runner to re-run affected tests
    6. Signal Quality Gate Enforcer to check all 7 gates`,
  subagent_type: "coder",
  description: "Bug Fixer",
  run_in_background: true
})

// Agent 5: Quality Gate Enforcer
Task({
  prompt: `You are the Quality Gate Enforcer agent. Your mission:

    After each fix cycle, evaluate ALL 7 quality gates:

    GATE 1 — FUNCTIONAL (100% required):
    - All tests in the target context pass
    - No regressions in previously passing tests

    GATE 2 — BEHAVIORAL (100% of targeted scenarios):
    - Every Gherkin scenario that was targeted has a passing test
    - Spec-to-test mapping is complete (no unmapped scenarios)

    GATE 3 — COVERAGE (>=85% overall, >=95% critical paths):
    - Calculate path coverage for the context
    - Critical paths: authentication, payment, core workflows
    - Non-critical paths: preferences, history, settings

    GATE 4 — SECURITY (0 violations):
    - No hardcoded API keys, tokens, or secrets in test files
    - No hardcoded test credentials (use env vars or test fixtures)
    - Secure storage patterns used (no plaintext sensitive data)

    GATE 5 — ACCESSIBILITY (WCAG AA):
    - All interactive elements have accessible labels
    - Touch/click targets meet minimum size requirements
    - Color contrast meets WCAG AA ratios
    - Screen reader navigation order is logical

    GATE 6 — RESILIENCE (tested for target context):
    - Offline/disconnected state handled gracefully
    - Timeout handling shows user-friendly message
    - Error states show retry option
    - Server errors show generic error, not stack trace

    GATE 7 — CONTRACT (0 mismatches):
    - API responses match expected schemas
    - No unexpected null fields
    - Enum values match expected set
    - Pagination format is consistent

    For each gate:
    - Status: PASS / FAIL / SKIP (with reason)
    - Details: what passed, what failed
    - Blocking: whether this gate blocks the commit

    Store gate results:
    npx @claude-flow/cli@latest memory store \
      --key "gates-[context]-[timestamp]" \
      --value "[gate results JSON]" \
      --namespace forge-state

    ONLY signal Auto-Committer when ALL 7 GATES PASS.`,
  subagent_type: "reviewer",
  description: "Quality Gate Enforcer",
  run_in_background: true
})

// Agent 6: Accessibility Auditor
Task({
  prompt: `You are the Accessibility Auditor agent. Your mission:

    1. For each screen/page/component in the target context, audit:

    LABELS:
    - Every interactive element has an accessible label/aria-label/Semantics label
    - Labels are descriptive (not "button1" but "Submit payment")
    - Images have alt text or semantic labels

    TOUCH/CLICK TARGETS:
    - All interactive elements meet minimum size (48x48dp mobile, 44x44px web)
    - Flag any undersized targets

    CONTRAST:
    - Text on colored backgrounds meets WCAG AA ratio (4.5:1 normal, 3:1 large)
    - Flag low-contrast combinations

    SCREEN READER:
    - Accessibility tree has logical reading order
    - No duplicate or misleading labels
    - Form fields have associated labels

    FOCUS/TAB ORDER:
    - Focus order follows visual layout
    - Focus trap in modals/dialogs
    - Focus returns to trigger after dialog closes

    2. Generate findings as:
       {severity: "critical"|"warning"|"info", element, file, line, issue, fix}

    3. Store audit results:
       npx @claude-flow/cli@latest memory store \
         --key "a11y-[context]-[timestamp]" \
         --value "[audit JSON]" \
         --namespace forge-state`,
  subagent_type: "analyst",
  description: "Accessibility Auditor",
  run_in_background: true
})

// Agent 7: Auto-Committer
Task({
  prompt: `You are the Auto-Committer agent. Your mission:

    1. Monitor for successful fixes where ALL 7 QUALITY GATES PASS
    2. For each successful fix:
       - Stage only the fixed files (never git add -A)
       - Create detailed commit message:

         fix(forge): Fix [TEST_ID] - [brief description]

         Behavioral Spec: [Gherkin scenario name]
         Root Cause: [what caused the failure]
         - [specific issue 1]
         - [specific issue 2]

         Fix Applied:
         - [change 1]
         - [change 2]

         Quality Gates:
         - Functional: PASS
         - Behavioral: PASS
         - Coverage: [X]%
         - Security: PASS
         - Accessibility: PASS
         - Resilience: PASS
         - Contract: PASS

         Confidence Tier: [platinum|gold|silver|bronze]
         Pattern Stored: fix-[error-type]-[hash]

       - Commit with the message above
    3. Update coverage report with new passing paths
    4. Store commit hash in memory for rollback capability:
       npx @claude-flow/cli@latest memory store \
         --key "commit-[hash]" \
         --value "[commit details JSON]" \
         --namespace forge-commits
    5. Store last known good commit:
       npx @claude-flow/cli@latest memory store \
         --key "last-green-commit" \
         --value "[hash]" \
         --namespace forge-state`,
  subagent_type: "reviewer",
  description: "Auto-Committer",
  run_in_background: true
})

// Agent 8: Learning Optimizer
Task({
  prompt: `You are the Learning Optimizer agent. Your mission:

    1. After each test cycle, analyze patterns:
       - Which error types fail most often?
       - Which fix patterns have highest success rate?
       - What new defensive patterns should be added?
       - Which Gherkin scenarios are most fragile?

    2. UPDATE CONFIDENCE TIERS:
       For each fix pattern applied this cycle:
       - If fix succeeded: confidence += 0.05 (cap at 1.0)
         - If confidence crosses 0.95: promote to Platinum
         - If confidence crosses 0.85: promote to Gold
       - If fix failed: confidence -= 0.10 (floor at 0.0)
         - If confidence drops below 0.70: demote to Bronze (learning-only)
       Store updated pattern:
       npx @claude-flow/cli@latest memory store \
         --key "fix-[error-type]-[hash]" \
         --value "[updated pattern JSON]" \
         --namespace forge-patterns

    3. DEFECT PREDICTION:
       Analyze which contexts/files are likely to fail next:
       - Files changed since last green run
       - Historical failure rate per context
       - Complexity of recent changes
       Store prediction:
       npx @claude-flow/cli@latest memory store \
         --key "prediction-[date]" \
         --value "[prediction JSON]" \
         --namespace forge-predictions

    4. Train neural patterns on successful fixes:
       npx @claude-flow/cli@latest hooks post-task \
         --task-id "forge-cycle" --success true --store-results true

    5. Update coverage status:
       npx @claude-flow/cli@latest memory store \
         --key "forge-coverage-status" \
         --value "[updated coverage JSON]" \
         --namespace forge-state

    6. Generate recommendations for test improvements
    7. Export learning metrics:
       npx @claude-flow/cli@latest neural train --pattern-type forge-fixes --epochs 5`,
  subagent_type: "researcher",
  description: "Learning Optimizer",
  run_in_background: true
})
```

---

## PHASE 5: QUALITY GATES

7 gates evaluated after each fix cycle. ALL must pass before a commit is created.

| Gate | Check | Threshold | Blocking |
|------|-------|-----------|----------|
| 1. Functional | All tests pass | 100% pass rate | YES |
| 2. Behavioral | Gherkin scenarios satisfied | 100% of targeted scenarios | YES |
| 3. Coverage | Path coverage | >=85% overall, >=95% critical | YES (critical only) |
| 4. Security | No hardcoded secrets, secure storage | 0 violations | YES |
| 5. Accessibility | Accessible labels, target sizes, contrast | WCAG AA | Warning only |
| 6. Resilience | Offline handling, timeout handling, error states | Tested for target context | Warning only |
| 7. Contract | API response matches expected schema | 0 mismatches | YES |

### Gate Failure Categories

When gates fail, failures are categorized for targeted re-runs:

- **Functional failures** → Re-run Bug Fixer on failing tests
- **Behavioral failures** → Check spec-to-test mapping, may need new tests
- **Coverage failures** → Generate additional test paths
- **Security failures** → Fix hardcoded values, update storage patterns
- **Accessibility failures** → Add accessible labels, fix target sizes
- **Resilience failures** → Add offline/error state handling
- **Contract failures** → Update DTOs or flag API regression

---

## AUTONOMOUS EXECUTION LOOP

```
┌────────────────────────────────────────────────────────────────────────┐
│                      FORGE AUTONOMOUS LOOP                             │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐        │
│  │ Specify  │───▶│   Test   │───▶│ Analyze  │───▶│   Fix    │        │
│  │ (Gherkin)│    │ (Run)    │    │ (Root    │    │ (Tiered) │        │
│  └──────────┘    └──────────┘    │  Cause)  │    └──────────┘        │
│       ▲                          └──────────┘         │               │
│       │                                               ▼               │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐        │
│  │  Learn   │◀───│  Commit  │◀───│  Gate    │◀───│  Audit   │        │
│  │ (Update  │    │ (Auto)   │    │ (7 Gates)│    │ (A11y)   │        │
│  │  Tiers)  │    └──────────┘    └──────────┘    └──────────┘        │
│  └──────────┘                                                         │
│       │                                                                │
│       └───────────────── REPEAT ──────────────────────────────────────│
│                                                                        │
│  Loop continues until: ALL 7 GATES PASS or MAX_ITERATIONS (10)        │
│  Gate failures are categorized for targeted re-runs (not full re-run) │
└────────────────────────────────────────────────────────────────────────┘
```

---

## CONFIDENCE TIERS FOR FIX PATTERNS

Every fix pattern is tracked with a confidence score that evolves over time:

```json
{
  "key": "fix-element-not-found-abc123",
  "pattern": {
    "error": "Element not found / No element",
    "fix": "Ensure element is rendered and visible before interaction",
    "files_affected": ["*_test.*"],
    "context": "any"
  },
  "tier": "gold",
  "confidence": 0.92,
  "auto_apply": true,
  "applied_count": 47,
  "success_count": 43,
  "success_rate": 0.915,
  "last_applied": "2026-02-06T14:30:00Z",
  "last_failed": "2026-02-01T09:15:00Z"
}
```

### Tier Thresholds

| Tier | Confidence | Auto-Apply | Behavior |
|------|-----------|------------|----------|
| **Platinum** | >= 0.95 | Yes | Apply immediately without review |
| **Gold** | >= 0.85 | Yes | Apply and flag in commit message |
| **Silver** | >= 0.75 | No | Suggest to Bug Fixer, don't auto-apply |
| **Bronze** | >= 0.70 | No | Store for learning only, never auto-apply |
| **Expired** | < 0.70 | No | Pattern demoted, needs revalidation |

### Confidence Updates

After each application:
- **Success:** confidence += 0.05 (capped at 1.0)
- **Failure:** confidence -= 0.10 (floored at 0.0)
- **Tier promotion** when crossing threshold upward
- **Tier demotion** when crossing threshold downward

---

## DEFECT PREDICTION

Before running tests, the Learning Optimizer analyzes historical data to predict which tests are most likely to fail:

### Input Signals

1. **Files changed** since last green run (git diff against last-green-commit)
2. **Historical failure rates** per bounded context (from forge-results namespace)
3. **Fix pattern freshness** — recently applied fixes are more likely to regress
4. **Complexity metrics** — contexts with more cyclomatic paths fail more often
5. **Dependency chain length** — deeper dependency chains have higher failure rates

### Prediction Output

```json
{
  "date": "2026-02-07",
  "predictions": [
    { "context": "payments", "probability": 0.73, "reason": "3 files changed in payment module" },
    { "context": "orders", "probability": 0.45, "reason": "depends on payments (changed)" },
    { "context": "identity", "probability": 0.12, "reason": "no changes, stable history" }
  ],
  "recommended_order": ["payments", "orders", "identity"]
}
```

Tests are executed in descending probability order — predicted-to-fail tests run FIRST for faster convergence.

---

## EXHAUSTIVE EDGE CASE TESTING

### General UI Element Edge Cases
For EVERY interactive element, test:

1. **Interaction States**
   - Single interaction → expected action
   - Repeated rapid interaction → no duplicate action
   - Long press / right-click → context menu if applicable
   - Disabled state → no action, visual feedback

2. **Input Field States**
   - Empty → placeholder visible
   - Focus → visual focus indicator
   - Valid input → no error
   - Invalid input → error message
   - Max length reached → prevents further input
   - Paste → validates pasted content
   - Clear → resets to empty

3. **Async Operation States**
   - Before load → loading indicator
   - During load → spinner, disabled submit
   - Success → data displayed, spinner gone
   - Error → error message, retry option
   - Timeout → timeout message, retry option

4. **Navigation Edge Cases**
   - Back navigation → previous screen or exit confirmation
   - Deep link → correct screen with params
   - Invalid deep link → fallback/error screen
   - Browser forward/back (web) → correct state

5. **Scroll Edge Cases**
   - Overscroll → appropriate feedback
   - Scroll to hidden content → content becomes visible
   - Keyboard appears → scroll to focused field

### Network Edge Cases
1. **No internet** → offline indicator, cached data if available
2. **Slow connection** → loading states persist, timeout handling
3. **Connection restored** → auto-retry pending operations
4. **Server error 500** → generic error message
5. **Auth error 401** → redirect to login
6. **Permission error 403** → permission denied message
7. **Not found 404** → "not found" message

### Chaos Testing (Resilience)

For each target context, inject controlled failures:

1. **Timeout injection** → API calls take >10s → verify timeout UI
2. **Partial response** → API returns incomplete data → verify graceful degradation
3. **Rate limiting** → API returns 429 → verify retry-after behavior
4. **Concurrent mutations** → Multiple clients modify same resource → verify conflict handling
5. **Session expiry** → Token expires mid-flow → verify re-auth prompt

---

## INVOCATION MODES

```bash
# Full autonomous run — all contexts, all gates
/forge --autonomous --all

# Single context autonomous
/forge --autonomous --context [context-name]

# Behavioral verification only (no fixes)
/forge --verify-only
/forge --verify-only --context [context-name]

# Fix-only mode (fix failures, don't generate new tests)
/forge --fix-only --context [context-name]

# Learn mode (analyze patterns, update confidence tiers)
/forge --learn

# Add coverage for new screens/pages/components
/forge --add-coverage --screens [name1],[name2]

# Generate Gherkin specs for a context
/forge --spec-gen --context [context-name]
/forge --spec-gen --all

# Run quality gates without test execution
/forge --gates-only
/forge --gates-only --context [context-name]

# Defect prediction only
/forge --predict
/forge --predict --context [context-name]

# Chaos/resilience testing for a context
/forge --chaos --context [context-name]
/forge --chaos --all
```

---

## MEMORY NAMESPACES

| Namespace | Purpose | Key Pattern |
|-----------|---------|-------------|
| `forge-patterns` | Fix patterns with confidence tiers | `fix-[error-type]-[hash]` |
| `forge-results` | Test run results | `test-run-[timestamp]` |
| `forge-state` | Coverage + gate status | `forge-coverage-status`, `gates-[context]-[ts]`, `last-green-commit` |
| `forge-commits` | Commit history | `commit-[hash]` |
| `forge-screens` | Implemented screens/pages | `screen-[name]` |
| `forge-specs` | Gherkin specifications | `specs-[context]-[timestamp]` |
| `forge-contracts` | API contract snapshots | `contract-snapshot-[timestamp]` |
| `forge-predictions` | Defect prediction history | `prediction-[date]` |

---

## DEFENSIVE TEST PATTERNS

The Bug Fixer agent uses defensive patterns appropriate to the project's test framework. Examples:

### Flutter: Safe Tap
```dart
Future<bool> safeTap(WidgetTester tester, Finder finder) async {
  await tester.pumpAndSettle();
  final elements = finder.evaluate();
  if (elements.isNotEmpty) {
    await tester.tap(finder.first, warnIfMissed: false);
    await tester.pumpAndSettle();
    return true;
  }
  debugPrint('Widget not found: ${finder.description}');
  return false;
}
```

### Flutter: Safe Text Entry
```dart
Future<bool> safeEnterText(WidgetTester tester, Finder finder, String text) async {
  await tester.pumpAndSettle();
  final elements = finder.evaluate();
  if (elements.isNotEmpty) {
    await tester.enterText(finder.first, text);
    await tester.pumpAndSettle();
    return true;
  }
  return false;
}
```

### Flutter: Visual Observation Delay
```dart
Future<void> visualDelay(WidgetTester tester, {String? label}) async {
  if (label != null) debugPrint('Observing: $label');
  await tester.pump(const Duration(milliseconds: 2500));
}
```

### Flutter: Scroll Until Visible
```dart
Future<bool> scrollUntilVisible(
  WidgetTester tester,
  Finder finder,
  Finder scrollable,
) async {
  for (int i = 0; i < 10; i++) {
    await tester.pumpAndSettle();
    if (finder.evaluate().isNotEmpty) return true;
    await tester.drag(scrollable, const Offset(0, -300));
    await tester.pumpAndSettle();
  }
  return false;
}
```

### Flutter: Wait For API Response
```dart
Future<void> waitForApiResponse(WidgetTester tester, {int maxWaitMs = 5000}) async {
  final startTime = DateTime.now();
  while (DateTime.now().difference(startTime).inMilliseconds < maxWaitMs) {
    await tester.pump(const Duration(milliseconds: 100));
    if (find.byType(CircularProgressIndicator).evaluate().isEmpty) break;
  }
  await tester.pumpAndSettle();
}
```

### Cypress / Playwright: Safe Click
```javascript
async function safeClick(selector, options = { timeout: 5000 }) {
  try {
    await page.waitForSelector(selector, { state: 'visible', timeout: options.timeout });
    await page.click(selector);
    return true;
  } catch (e) {
    console.warn(`Element not found: ${selector}`);
    return false;
  }
}
```

### Cypress / Playwright: Wait For API
```javascript
async function waitForApi(urlPattern, options = { timeout: 10000 }) {
  return page.waitForResponse(
    response => response.url().includes(urlPattern) && response.status() === 200,
    { timeout: options.timeout }
  );
}
```

---

## COMMON FIX PATTERNS

### Pattern: Element Not Found
```json
{
  "error": "Element not found / No element / Bad state: No element",
  "cause": "Element not rendered, wrong selector, or not in viewport",
  "tier": "platinum",
  "confidence": 0.97,
  "fixes": [
    "Wait for element to be rendered before interaction",
    "Use safe interaction helpers instead of direct calls",
    "Verify selector matches actual element",
    "Scroll element into view before interaction"
  ]
}
```

### Pattern: Timeout
```json
{
  "error": "Timeout / pumpAndSettle timed out / waiting for selector",
  "cause": "Infinite animation, continuous rebuild, or slow API",
  "tier": "gold",
  "confidence": 0.89,
  "fixes": [
    "Use fixed-duration wait instead of settle/idle wait",
    "Dispose animation controllers in tearDown",
    "Check for infinite re-render loops",
    "Increase timeout for slow API calls"
  ]
}
```

### Pattern: Assertion Failed
```json
{
  "error": "Expected: X, Actual: Y / AssertionError",
  "cause": "State not updated or wrong expectation",
  "tier": "silver",
  "confidence": 0.78,
  "fixes": [
    "Add delay before assertion for async state updates",
    "Verify test data seeding completed",
    "Check async operation completion before asserting"
  ]
}
```

### Pattern: API Response Mismatch
```json
{
  "error": "Type error / null value / schema mismatch",
  "cause": "Backend response format changed",
  "tier": "gold",
  "confidence": 0.86,
  "fixes": [
    "Update model/DTO to match current API response",
    "Add null safety handling",
    "Check API version compatibility"
  ]
}
```

---

## COVERAGE TRACKING

The Learning Optimizer maintains coverage status per context:

```json
{
  "lastRun": "2026-02-07T11:00:00Z",
  "backendStatus": {
    "healthy": true,
    "port": 8080
  },
  "gateStatus": {
    "functional": "PASS",
    "behavioral": "PASS",
    "coverage": "PASS",
    "security": "PASS",
    "accessibility": "WARNING",
    "resilience": "PASS",
    "contract": "PASS"
  },
  "contexts": {
    "[context-a]": { "total": 68, "passing": 68, "failing": 0, "behavioralCoverage": 100 },
    "[context-b]": { "total": 72, "passing": 70, "failing": 2, "behavioralCoverage": 97 }
  },
  "totalPaths": 0,
  "passingPaths": 0,
  "coveragePercent": 0,
  "confidenceTiers": {
    "platinum": 0,
    "gold": 0,
    "silver": 0,
    "bronze": 0,
    "expired": 0
  }
}
```

---

## AUTO-COMMIT MESSAGE FORMAT

```
fix(forge): Fix [TEST_ID] - [brief description]

Behavioral Spec: [Gherkin scenario name]
Root Cause: [what caused the failure]
- [specific issue 1]
- [specific issue 2]

Fix Applied:
- [change 1]
- [change 2]

Quality Gates:
- Functional: PASS
- Behavioral: PASS
- Coverage: [X]%
- Security: PASS
- Accessibility: PASS/WARNING
- Resilience: PASS
- Contract: PASS

Test Verification:
- Test now passes after fix
- No regression in related tests
- Dependent contexts re-tested: [list]

Confidence Tier: [platinum|gold|silver|bronze]
Pattern Stored: fix-[error-type]-[hash]
```

---

## ROLLBACK CAPABILITY

If a fix introduces regressions:

```bash
# Retrieve last known good commit
npx @claude-flow/cli@latest memory retrieve --key "last-green-commit" --namespace forge-state

# Rollback to that commit
git revert [bad-commit-hash]

# Store rollback event for learning (prevents pattern from being re-applied)
npx @claude-flow/cli@latest memory store \
  --key "rollback-[timestamp]" \
  --value '{"commit":"[hash]","reason":"[reason]","pattern":"[pattern-key]"}' \
  --namespace forge-patterns

# Demote the fix pattern confidence (-0.10)
# Learning Optimizer will handle this automatically
```

---

## POST-EXECUTION LEARNING

After each autonomous run, the skill triggers comprehensive learning:

```bash
# Train on successful patterns
npx @claude-flow/cli@latest hooks post-task --task-id "forge-run" --success true --store-results true

# Update neural patterns
npx @claude-flow/cli@latest neural train --pattern-type forge-fixes --epochs 5

# Update defect predictions
npx @claude-flow/cli@latest memory store \
  --key "prediction-$(date +%Y-%m-%d)" \
  --value "[prediction JSON from Learning Optimizer]" \
  --namespace forge-predictions

# Export metrics
npx @claude-flow/cli@latest hooks metrics --format json
```

---

## PROJECT-SPECIFIC EXTENSIONS

Forge can be extended per-project by creating a `forge.contexts.yaml` file alongside the skill:

```yaml
# forge.contexts.yaml — Project-specific bounded contexts and screens
contexts:
  - name: identity
    testFile: click_through_identity_full_test.dart
    specFile: identity.feature
    paths: 68
    subdomains: [Auth, Profiles, Verification]
    screens:
      - name: Identity Verification
        file: lib/screens/compliance/identity_verification_screen.dart
        route: /verification
        cyclomaticPaths:
          - All verifications incomplete -> show progress 0%
          - Email only verified -> show 25%
          - All verified -> show 100% + celebration state

  - name: payments
    testFile: click_through_payments_test.dart
    specFile: payments.feature
    paths: 89
    subdomains: [Wallet, Cards, Transactions]

dependencies:
  identity:
    blocks: [rides, payments, driver]
  payments:
    depends_on: [identity]
    blocks: [rides, subscriptions]
```

This separates the generic Forge engine from project-specific configuration, making Forge reusable across any codebase.

---

## QUICK REFERENCE CHECKLIST

Before running Forge:
- [ ] Backend built and running
- [ ] Health check passes
- [ ] Test data seeded via real API calls
- [ ] No mocking or stubbing in test code
- [ ] Gherkin specs exist for target context (or will be generated)
- [ ] All new screens/pages have test coverage
- [ ] Edge cases documented and tested

After Forge completes:
- [ ] Gate 1 (Functional): All tests pass
- [ ] Gate 2 (Behavioral): All targeted Gherkin scenarios satisfied
- [ ] Gate 3 (Coverage): >=85% overall, >=95% critical paths
- [ ] Gate 4 (Security): No hardcoded secrets
- [ ] Gate 5 (Accessibility): WCAG AA compliance checked
- [ ] Gate 6 (Resilience): Offline/timeout/error states tested
- [ ] Gate 7 (Contract): API responses match expected schemas
- [ ] Confidence tiers updated for all applied fix patterns
- [ ] Defect predictions updated for next run
- [ ] All fixes committed with detailed messages
