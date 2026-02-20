---
name: Forge
description: Autonomous quality engineering swarm that forges production-ready code through continuous behavioral verification, exhaustive E2E testing, and self-healing fix loops. Combines DDD+ADR+TDD methodology with BDD/Gherkin specifications, 7 quality gates, defect prediction, chaos testing, and cross-context dependency awareness. Architecture-agnostic - works with monoliths, microservices, modular monoliths, and any bounded-context topology.
---

# Forge - Autonomous Quality Engineering Swarm

**Quality forged in, not bolted on.**

Forge is a self-learning, autonomous quality engineering swarm that unifies three approaches into one:

| Pillar | Source | What It Does |
|--------|--------|--------------|
| **Build** | DDD+ADR+TDD methodology | Structured development with quality gates, defect prediction, confidence-tiered fixes |
| **Verify** | BDD/Gherkin behavioral specs | Continuous behavioral verification - the PRODUCT works, not just the CODE |
| **Heal** | Autonomous E2E fix loop | Test → Analyze → Fix → Commit → Learn → Repeat |

**"DONE DONE"** means: the code compiles AND the product behaves as specified. Every Gherkin scenario passes. Every quality gate clears. Every dependency graph is satisfied.

---

## TOPOLOGICAL GOVERNANCE FOUNDATIONS

Forge's autonomous pipeline is governed by topological invariants from the theory of topological governance in autonomous software engineering. Each subsection defines a formal specification that agents MUST follow. For infrastructure-dependent computations (Blake3, HNSW, WASM), the specification is defined with a readiness marker - agents approximate the computation via structured reasoning until native runtime is available.

### 1.1 Sheaf-Theoretic Consistency Model

Bounded contexts form a topological space where each context $U_i$ is an open set. Quality gates produce local sections $s_i \in \mathcal{F}(U_i)$ over each context. Global consistency is verified via sheaf cohomology:

- **$H^0(\mathcal{F})$ = global sections** - Gate results that agree across all context overlaps. When $H^0$ is non-trivial, the swarm has achieved a globally consistent quality state.
- **$H^1(\mathcal{F}) \neq 0$ = inter-context inconsistency** - A non-zero first cohomology group signals that local gate passes cannot be reconciled globally. Example: context A's contract tests pass against schema v2, but context B's contract tests assume schema v1. Both pass locally; the system fails globally. **Action: If $H^1(X;\, \mathcal{F}) \neq 0$, REJECT the commit immediately. The codebase state cannot be glued consistently.**

**Restriction maps:** For contexts $U_i \supseteq U_j$, the restriction $\rho_{ij}: \mathcal{F}(U_i) \to \mathcal{F}(U_j)$ is the projection of gate results onto the sub-context. Gate Enforcer verifies the cocycle condition: $\rho_{ij} \circ \rho_{jk} = \rho_{ik}$ on triple overlaps.

**Mathematical anchors** (Betti numbers): $\beta_0$ = number of connected components in the context dependency graph (should be 1 for a well-connected system). $\beta_1$ = number of independent cycles (each cycle is a potential inconsistency loop requiring explicit contract validation).

**Operational mapping:** Forge's cross-context dependency validation (Phase 2) and cascade re-testing IS the computation of restriction maps. Gate 7 (Contract) failures on shared types ARE non-zero H^1 elements.

### 1.2 Sheaf Laplacian & Dirichlet Energy

The discrete Sheaf Laplacian $L_{\mathcal{F}}$ is defined on the context dependency graph $G = (V, E)$ where V = bounded contexts and E = dependency edges:

$$L_{\mathcal{F}} = D_{\mathcal{F}} - A_{\mathcal{F}}$$

where $D_{\mathcal{F}}$ is the degree matrix weighted by restriction map norms and $A_{\mathcal{F}}$ is the adjacency matrix weighted by inter-context agreement.

**Dirichlet energy** quantifies total system tension:

$$E(S) = \sum_{(i,j) \in E} \left\|\rho_{ij}(s_i) - \rho_{ji}(s_j)\right\|^2$$

where $s_i$ is context i's gate result vector and $\rho_{ij}$ is the restriction map from context i to the shared boundary with context j.

**Operational mapping:** Forge's criticality score IS a discretized Dirichlet energy - the weighted combination of duration, blocking impact, cost, and detection rate across the agent graph measures the same "tension" that the Sheaf Laplacian formalizes. The stability threshold is $E(S) \leq 0.7$. If $E(S) > 0.7$, the Hallucination Gate MUST close. The agent's generation is blocked and a Blake3 witness is generated detailing the topological constraint violated.

### 1.3 Persistent Sheaf Laplacian

The commit history defines a filtration $\mathcal{F}_0 \subset \mathcal{F}_1 \subset \cdots \subset \mathcal{F}_n$ where each $\mathcal{F}_t$ is the codebase state at commit t. The Persistent Sheaf Laplacian tracks how Dirichlet energy evolves across this filtration.

**Persistence barcodes** per Gherkin scenario:

| Bar Type | Meaning | Forge Classification |
|----------|---------|---------------------|
| **Long bar** (born early, still alive) | Scenario has been stable across many commits | **Stable** (10+ consecutive passes) |
| **Short bar** (born and dies quickly) | Scenario flickers between pass/fail | **Flaky** (alternating pass/fail) |
| **Died bar** (was alive, now dead) | Scenario was passing, now consistently fails | **Regressed** (was stable, now failing) |

**Operational mapping:** Forge's behavioral regression tracking - storing the last 50 results per scenario with stability scores (Stable/Flaky/Regressed) - IS the persistence diagram. The `first_failure_commit` field marks the birth of a homological feature (a new failure mode). The `stability_score` is the bar length normalized to [0, 1].

### 1.4 Hallucination Gate - Deterministic Binary Boundary

The Hallucination Gate is a 3-phase deterministic verification that runs BEFORE any LLM-as-Judge evaluation. It provides a binary PASS/FAIL boundary that cannot be fooled by probabilistic reasoning.

**Phase 1 - AST Symbol Resolution:**
Parse the fix diff's AST. Every referenced symbol (class, function, method, import, type) MUST resolve to an existing definition in the codebase or SDK. Unresolved symbols = immediate FAIL.

**Phase 2 - Contract Hash Verification:**
Compute SHA-256 hash of the API specification (OpenAPI/schema) before and after the fix. If the fix claims to be non-breaking but the contract hash changed, FAIL. Contract mutations require explicit declaration.

**Phase 3 - Internal Mocking Detection:**
Regex scan for `@patch`, `mock`, `stub`, `fake`, `spy` targeting internal module paths. Any match = immediate FAIL. This is Gate 4's existing deterministic check, elevated to the Hallucination Gate.

**Gate ordering:** Primary gate = Hallucination Gate (deterministic). Secondary gate = LLM-as-Judge (probabilistic). A fix MUST pass the deterministic gate before the probabilistic gate is consulted. This prevents the "circular validation trap" where an LLM evaluates another LLM's output.

**Operational mapping:** Bug Fixer's Self-Reflection Gate Step 3.5 dimension (e) "EXISTENCE CHECK" IS Phase 1. Gate 4's mocking detection IS Phase 3. This subsection elevates them to a formal pre-LLM boundary and adds Phase 2 (contract hash).

### 1.5 Blake3 Cryptographic Witness Chain

Every gate verdict produces a cryptographic witness record forming an append-only hash chain:

```json
{
  "witness_id": "w-[gate]-[timestamp]",
  "gate": "functional|behavioral|coverage|security|accessibility|resilience|contract",
  "input_hash": "SHA-256 of test inputs + source files evaluated",
  "output_hash": "SHA-256 of gate verdict + evidence",
  "verdict": "PASS|FAIL",
  "timestamp": "ISO-8601",
  "prev_witness_hash": "SHA-256 of previous witness record in chain",
  "chain_position": "integer"
}
```

The chain is append-only - no witness can be modified after creation. Each witness references the previous, forming a tamper-evident log. Any break in the chain (hash mismatch) invalidates all subsequent witnesses.

**Infrastructure Dependency:** Blake3 hashing algorithm
**Readiness:** SPECIFICATION - agents use SHA-256 for witness hashing; computation is approximated via structured reasoning until Blake3 native runtime is available.
**Activation:** When Blake3 runtime is detected, agents switch from SHA-256 to Blake3 for witness hashing.

Witness records are stored in the `forge-witnesses` memory namespace with key pattern `witness-[gate]-[timestamp]`.

### 1.6 Algebraic Connectivity & Spectral Analysis

The agent collaboration graph $G = (V, E)$ has 8 vertices (one per agent) and edges weighted by data flow volume between agents. The graph Laplacian $L = D - A$ has eigenvalues $0 = \lambda_1 \leq \lambda_2 \leq \cdots \leq \lambda_8$.

**Fiedler value $\lambda_2$** = algebraic connectivity of the swarm:

**Hard requirement:** $\lambda_2$ MUST remain strictly > 0. A zero Fiedler value means the graph is disconnected - agents cannot coordinate.

| $\lambda_2$ Range | Classification | Action |
|----------|---------------|--------|
| $\lambda_2 \geq 0.5$ | **Well-connected** | Swarm is healthy, agents communicate effectively |
| $0.1 \leq \lambda_2 < 0.5$ | **Weakly connected** | Monitor for emerging fragmentation |
| $0 < \lambda_2 < 0.1$ | **Near-fragmentation** | Warning - strengthen inter-agent data flow |
| $\lambda_2 \approx 0$ | **SWARM_FRAGMENTATION** | Instant MinCut isolation + forced synchronization event - agents are disconnected |

**Spectral analysis procedure:**
1. Construct adjacency matrix A from agent data flow (memory reads/writes between namespaces)
2. Compute degree matrix D = diag(row sums of A)
3. Compute Laplacian $L = D - A$
4. Extract $\lambda_2$ (second-smallest eigenvalue)
5. If $\lambda_2 \approx 0$, execute MinCut isolation of disconnected subgraph + forced synchronization event. Emit SWARM_FRAGMENTATION alert to Learning Optimizer

**Operational mapping:** Forge's criticality scoring and bottleneck detection IS spectral analysis of the agent graph - bottleneck detection identifies agents with disproportionate blocking impact, which corresponds to vertices whose removal would disconnect the graph (low algebraic connectivity).

### 1.7 Dynamic MinCut Isolation

When an agent produces anomalous output (e.g., Bug Fixer generates a fix that fails the Hallucination Gate 3+ times consecutively), the system computes MinCut(G, anomalous_agent, Auto-Committer) to determine the minimum set of edges to sever to prevent anomalous output from reaching the commit stage.

**Quarantine protocol:**
1. Agent's output is logged but NOT forwarded to downstream agents
2. Failure Analyzer receives a QUARANTINE_ALERT with the agent's last 3 outputs
3. Learning Optimizer demotes all patterns applied by the quarantined agent (-0.10 each)
4. After root cause resolution, agent is un-quarantined and re-enters the pipeline

**Operational mapping:** Forge's sequential pipeline topology naturally provides MinCut = 1 per agent - each agent can be isolated by severing its single output edge. The blocking gate architecture (Gate Enforcer blocks Auto-Committer) IS a MinCut isolation boundary.

### 1.8 Hyperbolic Memory Architecture

Agent knowledge is embedded in the Poincaré ball model $\mathbb{B}^d = \{x \in \mathbb{R}^d : \|x\| < 1\}$ where hierarchical code relationships are preserved by hyperbolic distance:

$$d_H(u, v) = \operatorname{arcosh}\!\left(1 + \frac{2\|u - v\|^2}{(1 - \|u\|^2)(1 - \|v\|^2)}\right)$$

This metric naturally represents code taxonomy: packages near the origin are high-level abstractions; leaves near the boundary are concrete implementations. Parent-child distances are short; cross-branch distances are exponentially large.

**HNSW (Hierarchical Navigable Small World) index** over Poincaré embeddings enables $O(\log n)$ similarity search across the knowledge base - finding the most relevant fix pattern for a novel failure in sub-millisecond time.

**Infrastructure Dependency:** Vector database with hyperbolic distance metric + HNSW index
**Readiness:** SPECIFICATION - agents follow the hierarchical namespace structure; retrieval is approximated via key-based lookups across 10 namespaces until native vector DB is available.
**Activation:** When HNSW-capable vector DB is detected (or AQE ReasoningBank is available), agents switch from key-based to vector-similarity retrieval.

**Operational mapping:** Forge's 10 memory namespaces (forge-patterns, forge-results, forge-state, forge-commits, forge-screens, forge-specs, forge-contracts, forge-predictions, forge-criticality, forge-witnesses) ARE a flat approximation of the Poincaré ball - each namespace represents a region of the knowledge space. The Intelligence Plane is realized when these namespaces are backed by hyperbolic embeddings.

### 1.9 GF(3) Triadic Validation

Pipeline phase transitions are governed by Galois field $\mathrm{GF}(3) = \{-1,\, 0,\, +1\}$ trit values where:

| GF(3) Trit | Role | Meaning |
|------------|------|---------|
| -1 | Generator | Agent that produces output (e.g., Bug Fixer generates a fix) |
| 0 | Coordinator | Agent that orchestrates flow (e.g., Gate Enforcer routes decisions) |
| +1 | Validator | Agent that verifies correctness (e.g., Test Runner validates behavior) |

**Conservation law:** For any interacting triad of agents, the GF(3) sum MUST equal 0 (mod 3). Every generation (-1) must be balanced by a validation (+1) through a coordinator (0). If sum ≠ 0, block the transition and generate Narya-proofs documenting the conservation violation.

**Phase mapping:**

| Phase | GF(3) Index | Must Complete Before |
|-------|-------------|---------------------|
| Plan | 0 | Specify |
| Specify | 1 | Test |
| Test | 2 | Analyze |
| Analyze | 3 | Fix |
| Fix | 4 | Gate |
| Gate | 5 | Commit |
| Commit | 6 | Learn |
| Learn | 7 | Next iteration |

**Operational mapping:** Forge's "Plan Before Execute" mandate and sequential pipeline IS the operational implementation of GF(3) conservation. The blocking gate architecture enforces that no phase can be skipped - each phase's output is the next phase's input. The Generator→Coordinator→Validator triad maps directly to Forge's Bug Fixer(-1)→Gate Enforcer(0)→Test Runner(+1) cycle: $-1 + 0 + 1 \equiv 0 \pmod{3}$.

### 1.10 Narya-Proofs - Counterfactual Verification

Every Bug Fixer fix generates a Narya-proof: a bidirectional type-checking artifact that proves the fix is both necessary and sufficient.

**Forward type-check:** Apply the fix → run targeted tests → all PASS. This proves the fix is sufficient (it resolves the failure).

**Backward type-check:** Remove the fix (revert) → run targeted tests → at least one FAIL. This proves the fix is necessary (without it, the failure persists).

**Valid Narya-proof:** forward = PASS AND backward = FAIL.

| Forward | Backward | Verdict | Interpretation |
|---------|----------|---------|---------------|
| PASS | FAIL | **VALID** | Fix is necessary and sufficient |
| PASS | PASS | **COINCIDENTAL** | Fix is not the actual cause - tests pass without it |
| FAIL | FAIL | **INSUFFICIENT** | Fix does not resolve the failure |
| FAIL | PASS | **IMPOSSIBLE** | Logical contradiction - investigate test flakiness |

**Infrastructure Dependency:** Automated bidirectional test execution with git stash/unstash
**Readiness:** SPECIFICATION - agents follow the forward+backward verification protocol; full automation requires git-level rollback integration.
**Activation:** When forge-witnesses namespace is active, Narya-proofs are stored as `narya-[fix-hash]` entries.

**Operational mapping:** Bug Fixer's "targeted test re-run after fix" IS the forward type-check. The backward type-check is the new formal requirement - it ensures fixes are not coincidental.

### 1.11 Sublinear Coverage via Johnson-Lindenstrauss

For large test suites (n > 1000 tests), the Johnson-Lindenstrauss lemma guarantees that random projection from n dimensions to $O(\log n)$ dimensions preserves pairwise distances within $(1 \pm \varepsilon)$ factor.

**Application:** Project n test cases onto $O(\log n)$ representative dimensions. Each dimension corresponds to a topological feature of the codebase (a module boundary, an API endpoint, a state machine transition). The representative subset covers the same topological features as the full suite with high probability.

**Projection:**

$$\text{representative\_count} = O\!\left(\frac{\log n}{\varepsilon^2}\right)$$

For n = 1000 tests and $\varepsilon = 0.1$: representative_count ≈ 70 tests (93% reduction).

**Infrastructure Dependency:** Johnson-Lindenstrauss random projection matrix
**Readiness:** SPECIFICATION - agents use defect prediction to prioritize tests (greedy approximation of JL projection); full JL computation requires matrix operations.
**Activation:** When WASM runtime is available, agents compute exact JL projections for test selection.

**Operational mapping:** Forge's defect prediction ordering (predicted-to-fail first) IS a greedy approximation of JL projection - it selects the tests most likely to cover novel failure modes, achieving sublinear convergence without computing the full projection matrix.

### 1.12 WASM/Rust Execution Plane

Deterministic verification tasks are specified as pure functions suitable for WASM/Rust compilation:

| Task | Input | Output | Pure |
|------|-------|--------|------|
| Blake3 witness hashing | byte[] | hash | Yes |
| Eigenvalue computation ($\lambda_2$) | adjacency matrix | float | Yes |
| GF(3) phase validation | phase states | valid/invalid | Yes |
| HNSW nearest-neighbor | query vector, index | top-k results | Yes |
| Contract hash comparison | spec_before, spec_after | same/changed | Yes |
| JL random projection | test matrix, target dim | projected matrix | Yes |

**Infrastructure Dependency:** WASM runtime (e.g., Wasmtime, Wasmer) with Rust toolchain
**Readiness:** SPECIFICATION - all tasks are defined as pure functions; agents execute equivalent logic via structured reasoning until WASM runtime is available.
**Activation:** When WASM runtime is detected, deterministic tasks are offloaded from LLM reasoning to compiled execution for guaranteed correctness and sub-millisecond latency.

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
  "contexts": ["identity", "orders", "payments", "..."],
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
# forge.config.yaml (optional - Forge auto-discovers if absent)
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

## MOCKING POLICY: EXTERNAL ONLY, NEVER INTERNAL

**RULE: Mock ONLY external services. NEVER mock internal code.**

All tests run against the REAL backend API. Internal services, repositories, controllers, and models are NEVER mocked. Only third-party services outside your system boundary may be mocked or stubbed.

**Production Evidence:** In production orchestra runs, 5/5 PR failures (100%) were traced to internal mocking violations. 5/5 PR successes (100%) used real implementations. (See Issues #24, #25)

### Allowed - External Services Only

These are outside your system boundary and may be mocked:

- **Payment processors:** Stripe, PayPal, Braintree
- **Cloud services:** Firebase, AWS, GCP, Azure
- **Communication:** Twilio, SendGrid, Mailgun
- **Third-party APIs:** Google Places, Plaid, OAuth providers
- **HTTP clients:** Dio, Axios, fetch (when calling external URLs)
- **Infrastructure:** File system, network layer, system clock

### Forbidden - Internal Code (NEVER Mock)

These are inside your system and must use real implementations:

- **Your own services:** UserService, OrderService, PaymentService, ApiService
- **Models & entities:** User, Order, Payment, any domain object
- **Repositories & data access:** UserRepository, OrderRepository
- **Controllers & providers:** Any application-layer code you wrote
- **AI-generated code:** Any code produced by Forge or other agents

### Testing Strategy by Layer

| Layer | Approach |
|-------|----------|
| **Integration tests** | Real services + in-memory database, mock only external APIs |
| **E2E/BDD tests** | Real backend running locally, real API calls, real database |
| **Contract tests** | Real API responses compared against expected schemas |

### Examples

```python
# GOOD: Mock external Stripe API
@patch("src.services.stripe_client.StripeClient.create_charge")
async def test_payment_flow(mock_stripe):
    mock_stripe.return_value = {"id": "ch_test", "status": "succeeded"}
    response = await client.post("/api/v1/payments", json=payment_data)
    assert response.status_code == 201  # Real service, real DB, mocked Stripe

# BAD: Mock internal OrderService - NEVER DO THIS
@patch("src.services.order_service.OrderService.create_order")  # ❌ VIOLATION
async def test_checkout(mock_order):
    mock_order.return_value = Order(id=1)  # Hides real integration bugs
    ...

# CORRECT: Real implementation with test database
async def test_checkout():
    response = await client.post("/api/v1/checkout", json=checkout_data)
    assert response.status_code == 201  # Real OrderService, real DB
    order = await db.get(Order, response.json()["data"]["id"])
    assert order is not None  # Verify real data flow
```

### Enforcement

- **Coverage Validator:** Scans test files for internal mocking patterns - flags as CRITICAL violation
- **Gate 4 (Security):** Includes internal mocking check - BLOCKS commit if detected
- **Auto-Committer:** Refuses to commit code containing internal mock patterns
- **Pattern:** Any `@patch`, `mock`, `stub`, `fake`, `spy` targeting internal module paths triggers a BLOCK

---

## MANDATORY: PLAN BEFORE EXECUTE

**Every Forge invocation MUST call `EnterPlanMode` before executing any tasks - no exceptions.**

Before any phase begins, Forge enters planning mode to establish:

1. **Task breakdown** - discrete units of work derived from the target context
2. **Scope boundaries** - what is in-scope vs. out-of-scope for this run
3. **Success criteria** - measurable outcomes mapped to the 7 quality gates
4. **Dependencies** - backend readiness, test data, external service stubs
5. **Strategy** - execution order, model routing, and iteration budget

**No task execution begins without an approved plan.** This applies to all invocation modes: full swarm, single-gate re-runs, and targeted fixes. The plan is the contract between Forge and the developer - it ensures alignment before autonomous work starts.

---


## PHASE 0: BACKEND SETUP (MANDATORY FIRST STEP)

**BEFORE ANY TESTING, the backend MUST be built, compiled, and running.**

This is the FIRST thing the skill does - no exceptions.

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
# Seed test data through REAL API - adapt to your project's seeding endpoint
curl -X POST http://localhost:${BACKEND_PORT}/${SEED_ENDPOINT} \
  -H "Content-Type: application/json" \
  -H "${TEST_AUTH_HEADER}" \
  -d '${SEED_PAYLOAD}'
```

---

## PHASE 1: BEHAVIORAL SPECIFICATION & ARCHITECTURE RECORDS

**Before testing, verify Gherkin specs and architecture decision records exist for the target bounded context.**

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

### Spec Drift Detection

Gherkin specs are the behavioral contract. When specs and implementation diverge, the product is broken regardless of whether tests pass. Forge detects three types of drift:

**1. Static Drift - Code paths without matching specs**

Parse Gherkin Given/When/Then steps and verify matching code paths exist in the implementation. Flag:
- Implementation paths with no corresponding scenario (untested behavior)
- Scenarios referencing code paths that no longer exist (stale specs)
- New API endpoints with no behavioral specification

**2. Contract Drift - API specs vs live responses**

Compare API contracts defined in Gherkin scenarios against actual API responses:
- Expected response fields vs actual response fields
- Expected status codes vs actual status codes
- Expected error formats vs actual error formats

**3. Behavioral Regression Tracking**

Store the last N results (default: 50) per scenario to detect regressions over time:

```json
{
  "scenario": "User can complete payment",
  "history": [true, true, true, true, false],
  "stability_score": 0.80,
  "consecutive_passes_before_fail": 4,
  "first_failure_commit": "abc123",
  "status": "REGRESSED"
}
```

- **Stable** (10+ consecutive passes): scenario is reliable
- **Flaky** (alternating pass/fail): scenario needs investigation
- **Regressed** (was stable, now failing): high-priority alert with commit correlation

**Drift Severity Levels:**

| Severity | Meaning | Action |
|----------|---------|--------|
| **BLOCKING** | Implementation exists with no spec, or spec references removed code | Must resolve before Gate 2 |
| **WARNING** | Contract field mismatch or flaky scenario detected | Report in gate results, investigate |
| **INFO** | Minor drift (e.g., spec wording vs implementation naming) | Log for review |

### Agent-Optimized ADR Generation

When Forge discovers a bounded context without an Architecture Decision Record, the Specification Verifier generates one. ADRs follow an agent-optimized format designed for machine consumption:

```markdown
# ADR-NNN: [Context] Architecture Decision

## Status
Proposed | Accepted | Deprecated | Superseded by ADR-XXX

## MUST
- [Explicit required behaviors with contract references]
- [Link to OpenAPI spec: /api/v1/[context]/openapi.json]
- [Required integration patterns]

## MUST NOT
- [Explicit forbidden patterns]
- [Anti-patterns to avoid]
- [Coupling violations]

## Verification
- Command: [command to verify this decision holds]
- Expected: [expected output or exit code]

## Dependencies
- Depends on: [list of upstream contexts with ADR links]
- Blocks: [list of downstream contexts with ADR links]
```

**ADR Storage:**
- ADRs are stored in `docs/decisions/` or the project-configured ADR directory
- Each bounded context has exactly one ADR
- ADRs are updated when contracts change or new dependencies are discovered
- The Specification Verifier agent includes ADR generation in its workflow

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

### Shared Types Validation

For bounded contexts that share dependencies, validate type consistency across context boundaries:

1. **Identify shared DTOs/models** - For each context, extract types used in API requests and responses
2. **Cross-reference types** - Compare DTOs between contexts that share dependencies (from the dependency graph)
3. **Flag type mismatches** - e.g., context A expects `userId: string` but context B sends `userId: number`
4. **Validate value objects** - Ensure value objects (email, money, address) follow consistent patterns across contexts
5. **Report violations** - Flag as pre-Gate warnings with specific file locations and expected vs actual types

```json
{
  "sharedTypeViolation": {
    "type": "UserId",
    "contextA": { "name": "payments", "file": "types/payment.ts", "definition": "string" },
    "contextB": { "name": "orders", "file": "types/order.ts", "definition": "number" },
    "severity": "error"
  }
}
```

### Cross-Cutting Foundation Validation

Verify cross-cutting concerns are consistent across all bounded contexts:

- **Auth patterns** - Same header format (`Authorization: Bearer <token>`), same token validation approach across all endpoints
- **Error response format** - All API endpoints return errors in the project's standard format (consistent structure, error codes, HTTP status codes)
- **Logging patterns** - Consistent log levels, structured format, and correlation IDs across contexts
- **Pagination format** - Consistent pagination parameters and response format across collection endpoints

Cross-cutting violations are reported as warnings before Gate evaluation begins.

### Dependency Graph

Bounded contexts have dependencies. When a fix touches context X, all contexts that depend on X must be re-tested.

```yaml
# Context Dependency Map - define in forge.config.yaml or auto-discover
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

## MODEL ROUTING

Forge routes each agent to the appropriate model tier based on task complexity, optimizing for cost without sacrificing quality:

| Agent | Model | Rationale |
|-------|-------|-----------|
| Specification Verifier | `sonnet` | Reads code + generates Gherkin - moderate reasoning |
| Test Runner | `haiku` | Structured execution, output parsing - low reasoning |
| Failure Analyzer | `sonnet` | Root cause analysis - moderate reasoning |
| Bug Fixer | `opus` | First-principles code fixes - high reasoning |
| Quality Gate Enforcer | `haiku` | Threshold comparison - low reasoning |
| Accessibility Auditor | `sonnet` | Code analysis + WCAG rules - moderate reasoning |
| Auto-Committer | `haiku` | Git operations, message formatting - low reasoning |
| Learning Optimizer | `sonnet` | Pattern analysis, prediction - moderate reasoning |

Projects can override model assignments in `forge.config.yaml`:

```yaml
# forge.config.yaml - Model routing overrides (optional)
model_routing:
  spec-verifier: sonnet
  test-runner: haiku
  failure-analyzer: sonnet
  bug-fixer: opus
  gate-enforcer: haiku
  accessibility-auditor: sonnet
  auto-committer: haiku
  learning-optimizer: sonnet
```

When no override is specified, the defaults above are used. This routing reduces token cost by ~60% compared to running all agents on the highest-tier model.

### Energy-Based Lane Routing

The static agent-to-model mapping above serves as the default. At runtime, Coherence Energy (E) dynamically refines routing by selecting the appropriate processing lane for each task:

| Lane | Energy Range | Processing | Latency | Description |
|------|-------------|------------|---------|-------------|
| **Reflex** | $E < 0.1$ | WASM engine / ruleset | < 1ms | Zero LLM calls - deterministic checks (threshold comparisons, hash validations, format checks) |
| **Retrieval** | $0.1 \leq E < 0.4$ | Haiku-tier + RAG | ~10ms | Pattern-matched responses with retrieval-augmented context from forge-patterns |
| **Heavy** | $0.4 \leq E < 0.7$ | Opus-tier deep analysis | ~100ms | First-principles reasoning for novel failures and complex fixes |
| **Escalation** | $E \geq 0.7$ | Pause swarm | Human review | Dirichlet energy exceeds stability threshold - swarm pauses and escalates to human |

**How it works:** The existing UpgradeModel/DowngradeModel recommendations in criticality scoring already approximate energy-based routing. This formalizes those heuristics: when criticality is low (E < 0.1), skip the LLM entirely; when criticality exceeds the Dirichlet stability threshold (E ≥ 0.7), stop autonomous operation.

**Lane selection rule:** For each agent task, compute Coherence Energy E from the criticality score. The lane determines the model tier regardless of the agent's static default - a Gate Enforcer task that normally runs on haiku will escalate to opus if $E \in [0.4, 0.7)$, or pause the swarm entirely if $E \geq 0.7$.

---

## PHASE 4: SPAWN AUTONOMOUS AGENTS

Claude Code MUST spawn these 8 agents in a SINGLE message with `run_in_background: true`:

```javascript
// Agent 1: Specification Verifier
Task({
  model: "sonnet",
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
       - Run drift analysis: static drift (code paths vs spec steps),
         contract drift (API schema vs spec expectations),
         behavioral regression (historical pass/fail trends)
    5. Create spec-to-test mapping:
       - Each Scenario name → test function name
       - Store mapping in memory for Test Runner
    6. Store results:
       npx @claude-flow/cli@latest memory store --key "specs-[context]-[timestamp]" \
         --value "[spec status JSON]" --namespace forge-specs

    CONSTRAINTS:
    - NEVER generate specs for code you haven't read
    - NEVER assume UI elements exist without checking implementation
    - NEVER create scenarios that duplicate existing coverage
    - NEVER modify existing test files - only spec files

    ACCEPTANCE:
    - Every implementation file has at least one Gherkin scenario
    - Spec-to-test mapping has zero unmapped entries
    - All generated scenarios follow Given/When/Then format
    - Results stored in forge-specs namespace

    Output: List of all Gherkin scenarios with their mapped test functions, and any gaps found.`,
  subagent_type: "researcher",
  description: "Spec Verification",
  run_in_background: true
})

// Agent 2: Test Runner
Task({
  model: "haiku",
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
         --namespace forge-results

    CONSTRAINTS:
    - NEVER skip failing tests
    - NEVER modify test code or source code
    - NEVER mock internal code or stub our own APIs (external services OK)
    - NEVER continue if backend health check fails

    ACCEPTANCE:
    - All test results stored in memory with structured format
    - Zero unparsed failures - every failure has testId, error, stackTrace, file, line
    - Predicted-to-fail tests executed first
    - Results include Gherkin scenario mapping for every test`,
  subagent_type: "tester",
  description: "Test Runner",
  run_in_background: true
})

// Agent 3: Failure Analyzer
Task({
  model: "sonnet",
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
    6.5. MaTTS (Memory-Aware Test-Time Scaling) - For failures with no matching pattern:
       Generate 3 parallel reasoning trajectories:
       a) FORWARD: Trace execution from input → failure point. What state diverged?
       b) BACKWARD: Start from the assertion failure → trace back to the divergence point
       c) COUNTERFACTUAL: "If this fix were applied, would the failure disappear?"
       Self-contrast analysis: Compare all 3 trajectories. Where do they agree = high-confidence
       root cause. Where they diverge = investigate further. If all 3 agree on a root cause,
       promote the analysis confidence by +0.10.
       This implements MaTTS parallel trajectory generation with self-contrast - memory-aware
       because each trajectory queries forge-patterns for historical context.
    7. Store analysis in memory for Bug Fixer:
       npx @claude-flow/cli@latest memory store \
         --key "analysis-[testId]-[timestamp]" \
         --value "[analysis JSON]" \
         --namespace forge-results

    CONSTRAINTS:
    - NEVER assume root cause without stack trace evidence
    - NEVER recommend fixes for passing tests
    - NEVER skip dependency graph impact analysis
    - NEVER override confidence tier thresholds

    ACCEPTANCE:
    - Every failure has a root cause category and affected file
    - Zero unanalyzed failures
    - Dependency impact documented for every failure
    - Pattern search executed for every error type`,
  subagent_type: "researcher",
  description: "Failure Analyzer",
  run_in_background: true
})

// Agent 4: Bug Fixer
Task({
  model: "opus",
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

       BRONZE or NO PATTERN (use priority order):
       1. PREFERRED: Real implementation + in-memory DB
          - Use the actual service/repository with a test database
          - No mocking of any internal code
          - Seed test data through real API calls
       2. ACCEPTABLE: Real implementation + external-only mocks
          - Mock ONLY third-party services (Stripe, Firebase, etc.)
          - All internal code paths exercised for real
       3. LAST RESORT: Contract-first approach
          - Define the expected API contract (OpenAPI/schema)
          - Implement against the contract
          - Validate with contract tests
       4. NEVER: Mock internal services
          - NEVER create mock classes for internal services
          - NEVER stub repositories, controllers, or domain objects
          - Production evidence: 0% success rate for internal mocking (Issue #25)

    3. After fixing, identify affected context:
       - Check dependency graph for cascade impacts
       - Flag dependent contexts for re-testing

    3.5. SELF-REFLECTION GATE - Before storing the fix, ask "What could go wrong?":
       Evaluate across 5 dimensions:
       a) COMPLETENESS: Are there TODOs, placeholders, or stub implementations?
       b) ERROR HANDLING: Are all async operations wrapped in try-catch? Are API
          failures handled gracefully with user-facing error messages?
       c) EDGE CASES: What happens with empty input, null values, very large data,
          concurrent access, or rapid repeated actions?
       d) CONTRACT ADHERENCE: Does the fix match the Gherkin spec exactly? Are field
          names consistent between frontend and backend?
       e) EXISTENCE CHECK: Does every widget, class, function, and import I used
          actually exist in the SDK/framework? (Production evidence: non-existent
          RadioGroup<T> widget crashed 3 core features - Issue #21)

       If ANY dimension fails:
       - Fix the issue before proceeding
       - Re-run the targeted test to verify
       - Log the self-reflection finding for Learning Optimizer

       "Compilation does not equal Correctness." - validate existence and behavior.

    3.6. DRIVER-OBSERVER ALGEBRAIC CONNECTIVITY:
       Bug Fixer (opus) is the Driver; LLM-as-Judge (sonnet) is the Observer.
       Track pair connectivity: λ₂(pair) = submissions_accepted / total_submissions.
       - λ₂(pair) ≥ 0.5: Healthy collaboration - Driver and Observer agree frequently
       - λ₂(pair) < 0.5: Divergence - Observer is rejecting too many fixes
       - If Observer rejects 3+ consecutive submissions:
         → Emit DECOUPLE_ALERT
         → Request fresh root-cause analysis from Failure Analyzer
         → Reset the fix approach from first principles (do not retry same strategy)
       This prevents the Driver from fixating on a failing approach while the Observer
       repeatedly flags the same issue - a form of pair-programming deadlock.

    4. Store the fix pattern with initial confidence:
       npx @claude-flow/cli@latest memory store \
         --key "fix-[error-type]-[hash]" \
         --value '{"pattern":"[fix]","confidence":0.75,"tier":"silver","applied":1,"successes":0}' \
         --namespace forge-patterns

    5. Signal Test Runner to re-run affected tests
    6. Signal Quality Gate Enforcer to check all 7 gates

    CONSTRAINTS:
    - NEVER change test assertions to make tests pass
    - NEVER modify Gherkin specs to match broken behavior
    - NEVER introduce new dependencies without flagging
    - NEVER apply fixes without reading both test file and source file

    ACCEPTANCE:
    - Every applied fix has a targeted test re-run result
    - Zero fixes without verification
    - Fix pattern stored with initial confidence score
    - Cascade impacts identified and flagged for re-testing`,
  subagent_type: "coder",
  description: "Bug Fixer",
  run_in_background: true
})

// Agent 5: Quality Gate Enforcer
Task({
  model: "haiku",
  prompt: `You are the Quality Gate Enforcer agent. Your mission:

    After each fix cycle, evaluate ALL 7 quality gates:

    GATE 1 - FUNCTIONAL (100% required):
    - All tests in the target context pass
    - No regressions in previously passing tests

    GATE 2 - BEHAVIORAL (100% of targeted scenarios):
    - Every Gherkin scenario that was targeted has a passing test
    - Spec-to-test mapping is complete (no unmapped scenarios)

    GATE 3 - COVERAGE (>=85% overall, >=95% critical paths):
    - Calculate path coverage for the context
    - Critical paths: authentication, payment, core workflows
    - Non-critical paths: preferences, history, settings

    GATE 4 - SECURITY (0 critical/high violations):
    - No hardcoded API keys, tokens, or secrets in test files
    - No hardcoded test credentials (use env vars or test fixtures)
    - Secure storage patterns used (no plaintext sensitive data)
    - No SQL injection vectors in dynamic queries
    - No XSS vectors in rendered output
    - No path traversal in file operations
    - Dependencies have no known critical CVEs (when lockfile available)
    - No internal mocking violations (@patch/@mock targeting internal modules - BLOCKS commit)
    - When AQE available: delegate to security-scanner for full SAST analysis

    GATE 5 - ACCESSIBILITY (WCAG AA):
    - All interactive elements have accessible labels
    - Touch/click targets meet minimum size requirements
    - Color contrast meets WCAG AA ratios
    - Screen reader navigation order is logical

    GATE 6 - RESILIENCE (tested for target context):
    - Offline/disconnected state handled gracefully
    - Timeout handling shows user-friendly message
    - Error states show retry option
    - Server errors show generic error, not stack trace

    GATE 7 - CONTRACT (0 mismatches):
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

    ONLY signal Auto-Committer when ALL 7 GATES PASS.

    BFT CONSENSUS MODEL:
    The 7 gates operate as Byzantine Fault Tolerant validators:
    - Consensus threshold: ≥5/7 gates must PASS for a non-blocking consensus
    - Blocking gates (1-Functional, 2-Behavioral, 4-Security, 7-Contract) retain VETO power -
      a single blocking gate FAIL overrides BFT consensus
    - Non-blocking gates (3-Coverage, 5-Accessibility, 6-Resilience) participate in BFT consensus -
      they contribute warnings but cannot unilaterally block
    - CRDT counters: each gate maintains a grow-only PASS/FAIL counter across iterations.
      Counters are monotonically increasing (append-only) - they cannot be decremented or reset.
      This ensures gate history is tamper-evident across the autonomous loop.

    CONSTRAINTS:
    - NEVER approve a commit with ANY blocking gate failure
    - NEVER lower thresholds below defined minimums
    - NEVER skip gate evaluation - all 7 gates must be assessed
    - NEVER mark a gate as PASS without evidence

    ACCEPTANCE:
    - Gate results stored in memory with PASS/FAIL/SKIP for all 7 gates
    - Every FAIL includes specific details of what failed
    - Every SKIP includes reason for skipping
    - Auto-Committer only signaled when all blocking gates pass`,
  subagent_type: "reviewer",
  description: "Quality Gate Enforcer",
  run_in_background: true
})

// Agent 6: Accessibility Auditor
Task({
  model: "sonnet",
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
         --namespace forge-state

    CONSTRAINTS:
    - NEVER skip interactive elements during audit
    - NEVER report false positives for decorative images
    - NEVER ignore focus/tab order analysis
    - NEVER apply fixes - only report findings for Bug Fixer

    ACCEPTANCE:
    - Every interactive element audited
    - Findings stored with severity, element, file, line, issue, fix
    - Zero unaudited interactive elements in target context
    - WCAG AA compliance level assessed for every screen`,
  subagent_type: "analyst",
  description: "Accessibility Auditor",
  run_in_background: true
})

// Agent 7: Auto-Committer
Task({
  model: "haiku",
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
         --namespace forge-state

    CONSTRAINTS:
    - NEVER use git add -A or git add .
    - NEVER commit without all 7 gates passing
    - NEVER amend previous commits
    - NEVER push to remote - only local commits

    ACCEPTANCE:
    - Commit message includes Behavioral Spec, Root Cause, Fix Applied, all 7 gate statuses
    - Only fixed files are staged (no unrelated files)
    - Commit hash stored in forge-commits namespace
    - Last green commit updated in forge-state namespace`,
  subagent_type: "reviewer",
  description: "Auto-Committer",
  run_in_background: true
})

// Agent 8: Learning Optimizer
Task({
  model: "sonnet",
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

    6.5. DISTILL - Complete the ReasoningBank cycle (RETRIEVE/JUDGE/DISTILL/CONSOLIDATE):
       For fix patterns with ≥10 successful applications:
       a) Extract the common structure across all successful instances
       b) Generalize: replace context-specific values with pattern variables
       c) Abstract generalizable reasoning via Low-Rank Adaptation (LoRA) - extracting the minimal parameter delta that captures the pattern
       d) Store as a DISTILLED entry with elevated confidence (+0.05 bonus)
       e) Link the distilled pattern to its source instances for traceability
       Example: 10 successful "element-not-found" fixes across different contexts
       → DISTILL into: "Always waitForElement before interaction on any async-rendered widget"
       This completes the 4-phase ReasoningBank cycle:
       - RETRIEVE: query forge-patterns for matching fix patterns (step 3)
       - JUDGE: evaluate success/failure and update confidence (step 2)
       - DISTILL: generalize high-success patterns into reusable LoRA-style abstractions (this step)
       - CONSOLIDATE: integrate via Elastic Weight Consolidation (EWC++) to prevent catastrophic forgetting of successful patterns - never delete, only demote (step 2)

    7. Export learning metrics:
       npx @claude-flow/cli@latest neural train --pattern-type forge-fixes --epochs 5

    CONSTRAINTS:
    - NEVER promote a pattern that failed in the current cycle
    - NEVER delete patterns - only demote below Bronze threshold
    - NEVER override confidence scores without evidence from test results
    - NEVER generate predictions without historical data

    ACCEPTANCE:
    - All applied patterns have updated confidence scores
    - Prediction stored for next run with context-level probabilities
    - Coverage status updated in forge-state namespace
    - Zero patterns promoted without success evidence`,
  subagent_type: "researcher",
  description: "Learning Optimizer",
  run_in_background: true
})
```

---

## META-EVALUATION: LLM-AS-JUDGE REVIEW

After the fix loop completes, a meta-evaluation step reviews the Bug Fixer's output using a different model perspective. This catches issues that the builder misses - production evidence shows LLM-as-Judge found 0.4% test coverage across 250+ files in 7 minutes (Issue #20), while Self-Reflection caught a non-existent widget crashing 3 features (Issue #21). Multiple approaches find different issues with multiplicative (not additive) value (Issue #22).

### Activation

- **Automatic:** When fix confidence is below Silver (< 0.75)
- **Manual:** `--meta-review` flag on any invocation mode
- **Always-on:** Configurable in `forge.config.yaml` with `meta_review: always`

### Rubric (5 Dimensions)

The judge model evaluates the Bug Fixer's output against:

| Dimension | PASS Criteria | FAIL Criteria |
|-----------|--------------|---------------|
| **Functional Completeness** | No TODOs, stubs, or placeholder implementations | Any TODO/FIXME/stub found |
| **Error Handling** | All async operations wrapped in try-catch, user-facing error messages | Missing error handling on any API call |
| **Contract Alignment** | Fix matches Gherkin spec exactly, field names consistent | Spec divergence or field name mismatch |
| **Existence Verification** | Every widget/class/import exists in SDK/framework | Any reference to non-existent API |
| **Test Quality** | Tests cover happy path, error path, and edge cases | Missing error or edge case coverage |

### Verdict

- **PASS:** All 5 dimensions satisfied - proceed to commit
- **FAIL:** Any dimension fails - return to Bug Fixer with specific feedback

### Output

```json
{
  "verdict": "FAIL",
  "dimensions": {
    "functional_completeness": { "status": "PASS", "evidence": "No TODOs found" },
    "error_handling": { "status": "FAIL", "evidence": "Missing try-catch on /api/v1/payments POST" },
    "contract_alignment": { "status": "PASS", "evidence": "All fields match spec" },
    "existence_verification": { "status": "PASS", "evidence": "All imports verified" },
    "test_quality": { "status": "PASS", "evidence": "12 tests covering 3 paths" }
  },
  "recommendation": "Add error handling for payment API call before re-submission"
}
```

### Anti-Echo-Chamber Guarantee

The LLM-as-Judge architecture provides a provable anti-echo-chamber property:

- **Provably different priors:** Bug Fixer (opus) and LLM-as-Judge (sonnet) use different model architectures with different training distributions. This guarantees their error modes are not identical.
- **Error independence:** If each observer has $p(\text{error}) < 0.5$, the probability that ALL observers make the same error is $p^N$ (exponentially decreasing). With N=2 (Driver + Observer), $P(\text{both wrong}) < 0.25$.
- **High-stakes escalation:** For fixes below Silver confidence (< 0.75), require 3 independent priors (Driver + Observer + Failure Analyzer re-analysis). This reduces $P(\text{all wrong}) < 0.125$.
- **Architectural diversity ceiling:** The multi-tier model routing (opus/sonnet/haiku) ensures that at least 2 distinct model architectures evaluate every fix before commit.

---

## PHASE 5: QUALITY GATES

7 gates evaluated after each fix cycle. ALL must pass before a commit is created.

| Gate | Check | Threshold | Blocking |
|------|-------|-----------|----------|
| 1. Functional | All tests pass | 100% pass rate | YES |
| 2. Behavioral | Gherkin scenarios satisfied | 100% of targeted scenarios | YES |
| 3. Coverage | Path coverage | >=85% overall, >=95% critical | YES (critical only) |
| 4. Security | No hardcoded secrets, secure storage, SAST checks | 0 critical/high violations | YES |
| 5. Accessibility | Accessible labels, target sizes, contrast | WCAG AA | Warning only |
| 6. Resilience | Offline handling, timeout handling, error states | Tested for target context | Warning only |
| 7. Contract | API response matches expected schema | 0 mismatches | YES |

### Prime Radiant - Continuous Verification Daemon

The 7 quality gates collectively form the **Prime Radiant**: a continuous verification daemon that evaluates code correctness incrementally within each iteration, not just as an end-of-cycle batch.

- **Streaming evaluation:** Gate results update as new evidence arrives. When Test Runner completes a test batch, Gate 1 (Functional) updates immediately - it does not wait for all tests to finish. This enables early termination on blocking failures.
- **Čech nerve:** The bounded context open covers {U_i} form a Čech nerve N(U). Each simplex in the nerve corresponds to a set of contexts with non-empty intersection (shared dependencies). Gate 7 (Contract) evaluates on every simplex - verifying that shared types are consistent across all context overlaps.
- **Inter-iteration cohomology:** Between iterations, the Learning Optimizer computes global cohomology H^*(N(U)) by analyzing gate results across all contexts. A non-zero H^1 triggers immediate commit rejection and cross-context contract re-validation.

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
│  │   Plan   │───▶│ Specify  │───▶│   Test   │───▶│ Analyze  │        │
│  │ (Approve)│    │ (Gherkin)│    │ (Run)    │    │ (Root    │        │
│  └──────────┘    └──────────┘    └──────────┘    │  Cause)  │        │
│       ▲                                          └──────────┘        │
│       │                                               │               │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐        │
│  │  Learn   │◀───│  Commit  │◀───│  Gate    │◀───│  Audit   │        │
│  │ (Update  │    │ (Auto)   │    │ (7 Gates)│    │ (A11y +  │        │
│  │  Tiers)  │    └──────────┘    └──────────┘    │  Fix)    │        │
│  └──────────┘                                    └──────────┘        │
│       │                                                                │
│       └───────────────── REPEAT ──────────────────────────────────────│
│                                                                        │
│  Plan → Specify → Test → Analyze → Audit + Fix → Gate → Commit → Learn│
│  Loop continues until: ALL 7 GATES PASS or MAX_ITERATIONS (10)        │
│  Gate failures are categorized for targeted re-runs (not full re-run) │
└────────────────────────────────────────────────────────────────────────┘
```

**SPARC Pipeline Mapping:** Forge's 8-phase pipeline is a refinement of the SPARC (Specification-Pseudocode-Architecture-Refinement-Completion) methodology:

| SPARC Phase | Forge Phases | Agents |
|-------------|-------------|--------|
| **S**pecification | Plan + Specify | Spec Verifier |
| **P**seudocode | Test (executable specs = pseudocode made concrete) | Test Runner |
| **A**rchitecture | Analyze (root cause reveals architectural assumptions) | Failure Analyzer |
| **R**efinement | Fix + Gate (iterative refinement until all gates pass) | Bug Fixer, Gate Enforcer, A11y Auditor |
| **C**ompletion | Commit + Learn (verified completion with knowledge capture) | Auto-Committer, Learning Optimizer |

Forge's pipeline is a REFINEMENT of SPARC - it decomposes each SPARC phase into operationally distinct agents with quality gates between phases, enabling autonomous iteration without human intervention.

---

## REAL-TIME PROGRESS REPORTING

Each agent emits structured progress events during execution for observability:

```json
{"agent": "spec-verifier", "event": "spec_generated", "context": "payments", "scenarios": 12}
{"agent": "test-runner", "event": "test_started", "context": "payments", "test": "user_can_pay"}
{"agent": "test-runner", "event": "test_completed", "context": "payments", "passed": 10, "failed": 2}
{"agent": "failure-analyzer", "event": "root_cause_found", "test": "user_can_pay", "cause": "timeout"}
{"agent": "bug-fixer", "event": "fix_applied", "file": "payments.ts", "confidence": 0.92}
{"agent": "gate-enforcer", "event": "gate_evaluated", "gate": "functional", "status": "PASS"}
{"agent": "auto-committer", "event": "committed", "hash": "abc123", "tests_fixed": 2}
{"agent": "learning-optimizer", "event": "pattern_updated", "pattern": "fix-timeout-xyz", "tier": "gold"}
```

**Progress File:**
- Events are appended to `.forge/progress.jsonl` (one JSON object per line)
- File is created at the start of each Forge run and truncated
- Tools can tail this file for real-time monitoring: `tail -f .forge/progress.jsonl`

**Integration with Agentic QE AG-UI:**
- When the AQE AG-UI protocol is available, events stream directly to the user interface
- Users see live progress: which gate is being evaluated, which test is running, which fix is being applied
- When running in Claude Code without AG-UI, progress is visible through agent output files

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

### Nash Equilibrium Property

The asymmetric update rule (+0.05 success / -0.10 failure) implements a Nash equilibrium in the pattern confidence game:

- **Break-even probability:** A pattern must succeed at least $P(\text{success}) \geq \frac{2}{3} \approx 0.667$ to maintain its confidence level. At exactly 2/3 success rate, expected confidence change per application = $\frac{2}{3}(+0.05) + \frac{1}{3}(-0.10) = 0$.
- **Bronze threshold (0.70)** is set just above the equilibrium point (0.667) - a pattern that barely breaks even cannot auto-apply. Only patterns with demonstrated reliability above equilibrium advance.
- **No incentive exploitation:** An agent cannot game the tier system by applying a pattern speculatively. The 2:1 penalty-to-reward ratio ensures that any strategy with < 67% success rate leads to demotion, making speculative application a losing strategy.
- **Convergence guarantee:** Patterns converge to their true success rate over time. High-quality patterns rise to Platinum; unreliable patterns sink to Expired. The equilibrium prevents oscillation.

---

## DEFECT PREDICTION

Before running tests, the Learning Optimizer analyzes historical data to predict which tests are most likely to fail:

### Input Signals

1. **Files changed** since last green run (git diff against last-green-commit)
2. **Historical failure rates** per bounded context (from forge-results namespace)
3. **Fix pattern freshness** - recently applied fixes are more likely to regress
4. **Complexity metrics** - contexts with more cyclomatic paths fail more often
5. **Dependency chain length** - deeper dependency chains have higher failure rates

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

Tests are executed in descending probability order - predicted-to-fail tests run FIRST for faster convergence.

---

## AGENT CRITICALITY & BOTTLENECK DETECTION

Forge continuously monitors agent performance to identify bottlenecks and optimize the orchestra. Each agent and quality gate receives a criticality score (0.0–1.0) that drives automatic optimization decisions.

### Criticality Score Formula

```
criticality = (duration_weight × normalized_duration)
            + (blocking_weight × blocking_impact)
            + (cost_weight × normalized_cost)
            + (detection_weight × issue_detection_rate)
```

| Factor | Weight | Description |
|--------|--------|-------------|
| **Duration** | 0.30 | Wall-clock time as fraction of total run |
| **Blocking Impact** | 0.30 | Number of downstream agents/gates blocked while waiting |
| **Model Cost** | 0.20 | Token cost as fraction of total run cost |
| **Issue Detection Rate** | 0.20 | Ratio of real issues found to total items checked |

### Bottleneck Thresholds

| Criticality | Classification | Action |
|-------------|---------------|--------|
| > 0.8 | **Critical bottleneck** | Immediate optimization required |
| $0.5$-$0.8$ | **Moderate bottleneck** | Optimization recommended |
| < 0.5 | **Healthy** | No action needed |

### Automatic Optimization Recommendations

When a bottleneck is detected, Forge recommends (and can auto-apply) these optimizations:

| Recommendation | When Applied | Example |
|---------------|-------------|---------|
| **AddParallelism** | Duration high, work is splittable | Run Test Runner per context in parallel |
| **UpgradeModel** | Detection rate low, cost is acceptable | Promote Failure Analyzer from sonnet → opus |
| **DowngradeModel** | Cost high, detection rate already high | Demote Gate Enforcer from sonnet → haiku |
| **ReorderExecution** | Blocking impact high | Move Gate 4 check earlier in the pipeline |
| **CacheResults** | Same analysis repeated across runs | Cache contract snapshots between runs |

### Metrics Storage

```json
{
  "run_id": "forge-2026-02-19-001",
  "agent_metrics": {
    "bug-fixer": { "duration_ms": 45000, "cost_tokens": 12000, "issues_found": 3, "criticality": 0.72 },
    "test-runner": { "duration_ms": 30000, "cost_tokens": 2000, "issues_found": 5, "criticality": 0.45 },
    "gate-enforcer": { "duration_ms": 5000, "cost_tokens": 800, "issues_found": 1, "criticality": 0.15 }
  },
  "bottlenecks": ["bug-fixer"],
  "recommendations": [
    { "agent": "bug-fixer", "action": "CacheResults", "reason": "Same contract validation repeated 3x" }
  ]
}
```

Metrics are stored in the `forge-criticality` namespace for trend analysis across runs.

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

### Visual Regression Testing

For UI-heavy projects, Forge captures and compares screenshots to detect unintended visual changes:

1. **Before fix** - Capture baseline screenshots of all screens in the target context
2. **After fix** - Capture new screenshots of the same screens
3. **Compare** - Pixel-by-pixel comparison with configurable threshold (default: 0.1% diff tolerance)
4. **Report** - Flag visual regressions as Gate 5 (Accessibility) warnings
5. **Store** - Save screenshot diffs in memory for review

**Screenshot Capture by Platform:**

| Platform | Method |
|----------|--------|
| Web (Playwright) | `page.screenshot({ fullPage: true })` |
| Web (Cypress) | `cy.screenshot()` |
| Flutter | `await tester.binding.setSurfaceSize(size); await expectLater(find.byType(App), matchesGoldenFile('name.png'))` |
| Mobile (native) | Platform-specific screenshot capture |

**Configuration:**

```yaml
# forge.config.yaml - Visual regression settings (optional)
visual_regression:
  enabled: true
  threshold: 0.001  # 0.1% pixel diff tolerance
  screenshot_dir: .forge/screenshots
  full_page: true
```

When Agentic QE is available, delegate to the `visual-tester` agent for parallel viewport comparison across multiple screen sizes.

### Property-Based Testing

Instead of writing individual test cases, define invariants that must hold for ALL inputs. Forge extracts invariants from Gherkin specs and ADRs, then generates 1000+ random test cases per invariant.

**Process:**

1. Extract invariants from Gherkin scenarios and ADRs (e.g., "balance is always >= 0")
2. Generate random inputs covering the input space (edge values, boundary conditions, random data)
3. Run each invariant against all generated inputs
4. On failure: automatically shrink to the minimal counterexample
5. Report the minimal failing case with full reproduction steps

**Framework Tools by Language:**

| Language | Library | Example |
|----------|---------|---------|
| Dart/Flutter | `check` | `forAll(integer(), (n) => balance(n) >= 0)` |
| JavaScript/TS | `fast-check` | `fc.assert(fc.property(fc.integer(), (n) => balance(n) >= 0))` |
| Python | `hypothesis` | `@given(st.integers()) def test_balance(n): assert balance(n) >= 0` |
| Rust | `proptest` / `quickcheck` | `proptest!(\|(n: i32)\| prop_assert!(balance(n) >= 0))` |
| Go | `rapid` | `rapid.Check(t, func(t *rapid.T) { n := rapid.Int().Draw(t, "n"); assert(balance(n) >= 0) })` |

**Invariant Sources:**

- Gherkin `Then` clauses that assert universal properties ("balance is never negative")
- ADR `MUST` constraints ("all prices must be positive")
- Domain rules from bounded context definitions

### Mutation Testing

Mutation testing verifies that your tests actually catch bugs by injecting deliberate mutations into critical code paths and checking whether tests detect them.

**Process:**

1. Identify critical code paths (payment calculations, auth flows, state machines)
2. Inject mutations: flip operators (`==` → `!=`), change constants, remove null checks, swap conditions
3. Run the test suite against each mutant
4. **Killed** = test caught the mutation (good). **Survived** = test missed it (gap found)
5. Report mutation score and surviving mutants

**Targets:**

| Mutation Score | Classification |
|---------------|---------------|
| > 85% | **Critical paths** - required for Gate 3 pass |
| > 70% | **Overall codebase** - recommended minimum |
| < 70% | **Insufficient** - test gaps exist |

**Example Mutations:**

```
Original:  if (amount > 0) { processPayment(amount); }
Mutant 1:  if (amount >= 0) { processPayment(amount); }  // boundary
Mutant 2:  if (amount < 0) { processPayment(amount); }   // negation
Mutant 3:  if (true) { processPayment(amount); }          // constant
Mutant 4:  // removed: processPayment(amount);             // deletion
```

If any mutant survives (tests still pass), a test gap exists and Forge generates a new test case to kill the mutant.

---

## INVOCATION MODES

```bash
# Full autonomous run - all contexts, all gates
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

# Spec drift detection
/forge --drift-check
/forge --drift-check --context [context-name]

# Behavioral regression analysis
/forge --regressions
/forge --regressions --context [context-name]

# LLM-as-Judge meta-review
/forge --meta-review
/forge --meta-review --context [context-name]

# Mutation testing
/forge --mutation --context [context-name]
/forge --mutation --critical-only
```

---

## MODE-SPECIFIC BEHAVIOR

Each invocation mode controls which agents spawn, which phases execute, and what output is produced. This section defines the exact behavior for modes that modify the default autonomous pipeline.

### `--verify-only`

**Purpose:** Validate that specs and tests pass without applying any fixes.

**Agents spawned:** Spec Verifier, Test Runner, Gate Enforcer
**Agents skipped:** Bug Fixer, Auto-Committer, Learning Optimizer, Failure Analyzer, A11y Auditor

**Execution:**

1. **Phase 1 (Plan):** Discover bounded contexts and load forge.config.yaml - same as autonomous
2. **Phase 2 (Specify):** Spec Verifier checks spec-to-test mapping completeness
3. **Phase 3 (Test):** Test Runner executes the full test suite for targeted context(s)
4. **Phase 4 (Fix):** SKIPPED - no fixes are generated or applied
5. **Phase 5 (Gate):** Gate Enforcer evaluates Gate 1 (Functional), Gate 2 (Behavioral), and Gate 7 (Contract)
6. **Phase 6 (Commit):** SKIPPED - no changes to commit
7. **Phase 7 (Learn):** SKIPPED - no fix patterns to record

**Output:** Pass/fail report per evaluated gate. No files are modified.

```json
{
  "mode": "verify-only",
  "context": "identity",
  "gates": {
    "functional": { "status": "PASS", "tests_run": 47, "tests_passed": 47 },
    "behavioral": { "status": "FAIL", "mapped": 12, "unmapped": 2, "unmapped_scenarios": ["User resets password via SMS", "Admin revokes session"] },
    "contract": { "status": "PASS", "endpoints_checked": 8, "mismatches": 0 }
  },
  "verdict": "FAIL",
  "reason": "Gate 2 (Behavioral) has 2 unmapped scenarios"
}
```

### `--drift-check`

**Purpose:** Detect divergence between Gherkin specs, API contracts, and implementation without running tests or applying fixes.

**Agents spawned:** Spec Verifier only
**Agents skipped:** Test Runner, Bug Fixer, Auto-Committer, Learning Optimizer, Failure Analyzer, A11y Auditor, Gate Enforcer

**Execution:**

1. **Phase 1 (Plan):** Discover bounded contexts and load forge.config.yaml - same as autonomous
2. **Phase 2 (Specify):** Spec Verifier executes all 3 drift detection types:
   - **Static Drift:** Parse Gherkin steps and verify matching code paths exist
   - **Contract Drift:** Compare API specs in Gherkin against actual endpoint definitions (OpenAPI specs or route declarations)
   - **Behavioral Regression:** Analyze stored scenario history for stability/flaky/regressed status
3. **Phases 3–7:** SKIPPED - no test execution, fixing, gating, committing, or learning

**Output:** Drift report with severity per finding. No files are modified.

```json
{
  "mode": "drift-check",
  "context": "payments",
  "findings": [
    {
      "type": "static",
      "severity": "BLOCKING",
      "description": "POST /api/v1/payments/refund exists in implementation but has no Gherkin scenario",
      "file": "src/payments/routes.rs",
      "line": 142
    },
    {
      "type": "contract",
      "severity": "WARNING",
      "description": "Gherkin expects field 'payment_id' but API returns 'id'",
      "spec": "specs/payments.feature:34",
      "endpoint": "GET /api/v1/payments/:id"
    },
    {
      "type": "behavioral_regression",
      "severity": "WARNING",
      "description": "Scenario 'User completes checkout' regressed after 10 consecutive passes",
      "scenario": "specs/payments.feature:48",
      "first_failure_commit": "abc123",
      "stability_score": 0.80
    }
  ],
  "summary": { "BLOCKING": 1, "WARNING": 2, "INFO": 0 }
}
```

### `--meta-review`

**Purpose:** Force LLM-as-Judge evaluation of Bug Fixer output regardless of confidence tier.

**Behavior:** Modifier flag that can be combined with other modes or used standalone.

**When combined with `--autonomous`:**
Forces the LLM-as-Judge meta-evaluation (5-dimension rubric) after every Bug Fixer cycle, regardless of confidence tier. Normally, meta-review activates only when fix confidence is below Silver (< 0.75). This flag overrides that threshold.

**When used standalone (`/forge --meta-review` or `/forge --meta-review --context [name]`):**

**Agents spawned:** Spec Verifier (for contract alignment check) + a judge-perspective model
**Agents skipped:** Test Runner, Bug Fixer, Auto-Committer, Learning Optimizer, Failure Analyzer, A11y Auditor

**Execution:**

1. **Phase 1 (Plan):** Discover bounded contexts and load forge.config.yaml
2. **Evaluate:** Apply the 5-dimension rubric (Functional Completeness, Error Handling, Contract Alignment, Existence Verification, Test Quality) against the most recent Bug Fixer output for the targeted context. If no prior output exists, evaluate the current test files and implementation code in the context.
3. **Phases 3–7:** SKIPPED - no test execution, fixing, gating, committing, or learning

**Output:** JSON verdict with dimension-level pass/fail per the META-EVALUATION rubric. No files are modified.

```json
{
  "mode": "meta-review",
  "context": "identity",
  "verdict": "FAIL",
  "dimensions": {
    "functional_completeness": { "status": "PASS", "evidence": "No TODOs or stubs found" },
    "error_handling": { "status": "FAIL", "evidence": "Missing try-catch on /api/v1/auth/refresh POST" },
    "contract_alignment": { "status": "PASS", "evidence": "All fields match Gherkin spec" },
    "existence_verification": { "status": "PASS", "evidence": "All imports and references verified" },
    "test_quality": { "status": "PASS", "evidence": "15 tests covering happy, error, and edge paths" }
  },
  "recommendation": "Add error handling for auth refresh endpoint before proceeding"
}
```

### `--mutation --critical-only`

**Purpose:** Run mutation testing scoped to critical code paths only, with a higher kill-rate threshold.

**Critical path definition:** Functions in code paths matching Gate 3's critical path categories - authentication, payment processing, and core state machines. Specifically:
- Authentication flows (login, signup, token refresh, session management)
- Payment processing (charge, refund, subscription lifecycle)
- Core state machines (order state transitions, booking lifecycle, workflow engines)

**Agents spawned:** Test Runner (for mutant execution) + Gate Enforcer (for threshold evaluation)
**Agents skipped:** Bug Fixer, Auto-Committer, Learning Optimizer, Spec Verifier, Failure Analyzer, A11y Auditor

**Execution:**

1. **Phase 1 (Plan):** Discover bounded contexts and load forge.config.yaml
2. **Identify critical paths:** Scan targeted context(s) for functions in authentication, payment, and core state machine modules. Use directory structure, module names, and forge.config.yaml `critical_paths` configuration (if defined) to identify scope.
3. **Mutate:** Inject mutations only into identified critical-path functions: flip operators (`==` → `!=`), change constants, remove null checks, swap conditions
4. **Execute:** Run the test suite against each mutant
5. **Evaluate:** Apply kill-rate threshold of **>=85%** (vs >=70% for full `--mutation`)
6. **Phases 6–7:** SKIPPED - no commits or learning

**Output:** Surviving mutants list with test gap recommendations. No files are modified.

```json
{
  "mode": "mutation-critical-only",
  "context": "payments",
  "critical_paths_identified": [
    "src/payments/charge.rs",
    "src/payments/refund.rs",
    "src/payments/subscription.rs"
  ],
  "mutants_generated": 42,
  "mutants_killed": 38,
  "mutants_survived": 4,
  "kill_rate": 0.905,
  "threshold": 0.85,
  "verdict": "PASS",
  "surviving_mutants": [
    {
      "file": "src/payments/refund.rs",
      "line": 67,
      "mutation": "Changed `amount > 0` to `amount >= 0`",
      "recommendation": "Add test for zero-amount refund rejection"
    },
    {
      "file": "src/payments/refund.rs",
      "line": 103,
      "mutation": "Removed null check on `transaction_id`",
      "recommendation": "Add test for nil transaction_id in refund request"
    },
    {
      "file": "src/payments/subscription.rs",
      "line": 45,
      "mutation": "Swapped `Active` → `Paused` in state transition",
      "recommendation": "Add test verifying subscription activation sets state to Active"
    },
    {
      "file": "src/payments/charge.rs",
      "line": 29,
      "mutation": "Changed `currency == 'USD'` to `currency != 'USD'`",
      "recommendation": "Add explicit currency validation test for USD charges"
    }
  ]
}
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
| `forge-criticality` | Agent performance metrics & bottleneck data | `criticality-[run-id]` |
| `forge-witnesses` | Blake3 witness chain + Narya-proofs | `witness-[gate]-[ts]`, `narya-[fix-hash]` |

---

## OPTIONAL: AGENTIC QE INTEGRATION

Forge can optionally integrate with the [Agentic QE](https://github.com/proffesor-for-testing/agentic-qe) framework via MCP for enhanced capabilities. **All AQE features are additive - Forge works identically without AQE.**

### Detection

On startup, Forge checks for AQE availability:

```bash
# Check if agentic-qe MCP server is registered
claude mcp list | grep -q "aqe" && echo "AQE available" || echo "AQE not available - using defaults"
```

### Enhanced Capabilities When AQE Is Available

| Forge Component | Without AQE (Default) | With AQE |
|----------------|----------------------|----------|
| **Pattern Storage** | claude-flow memory (`forge-patterns` namespace) | ReasoningBank - HNSW vector-indexed, 150x faster pattern search, experience replay |
| **Defect Prediction** | Historical failure rates + file changes | `defect-intelligence` domain - root-cause-analyzer + defect-predictor agents |
| **Security Scanning** | Gate 4 static checks (secrets, injection vectors) | `security-compliance` domain - full SAST/DAST via security-scanner agent |
| **Accessibility Audit** | Forge Accessibility Auditor agent | `visual-accessibility` domain - visual-tester + accessibility-auditor agents |
| **Contract Testing** | Gate 7 schema validation | `contract-testing` domain - contract-validator + graphql-tester agents |
| **Progress Reporting** | `.forge/progress.jsonl` file | AG-UI streaming protocol for real-time UI updates |

### Fallback Behavior

When AQE is NOT available, Forge falls back to its built-in behavior for every capability. No configuration is required - the skill auto-detects and adapts.

### Configuration

```yaml
# forge.config.yaml - AQE integration settings (optional)
integrations:
  agentic-qe:
    enabled: true  # auto-detected if not specified
    domains:
      - defect-intelligence
      - security-compliance
      - visual-accessibility
      - contract-testing
    reasoning_bank:
      enabled: true  # replaces claude-flow memory for forge-patterns namespace
    ag_ui:
      enabled: true  # stream progress events to AG-UI protocol
```

### AQE Agent Delegation Map

When AQE is enabled, Forge delegates specific subtasks to specialized AQE agents:

| Forge Agent | AQE Domain | AQE Agents Used |
|-------------|-----------|-----------------|
| Specification Verifier | `requirements-validation` | bdd-generator, requirements-validator |
| Failure Analyzer | `defect-intelligence` | root-cause-analyzer, defect-predictor |
| Quality Gate Enforcer (Gate 4) | `security-compliance` | security-scanner, security-auditor |
| Accessibility Auditor | `visual-accessibility` | visual-tester, accessibility-auditor |
| Quality Gate Enforcer (Gate 7) | `contract-testing` | contract-validator, graphql-tester |
| Learning Optimizer | `learning-optimization` | learning-coordinator, pattern-learner |

Forge agents that have no AQE equivalent (Test Runner, Bug Fixer, Auto-Committer) continue to run as built-in agents regardless of AQE availability.

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

## ROLLBACK & CONFLICT RESOLUTION

### Rollback Capability

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

### Fix Conflict Protocol

When Bug Fixer's fix causes a cascade regression (tests in dependent contexts fail):

1. **Halt** - Stop the fix loop for the affected context
2. **Re-analyze** - Failure Analyzer examines both the original failure AND the cascade failure
3. **Categorize** - Compare root cause categories:
   - **Different root cause** → The fix is kept; the cascade failure is treated as a new, independent failure in the next loop iteration
   - **Same root cause** → The fix is reverted and the pattern is demoted (-0.10 confidence)
4. **Revert limit** - Maximum 2 revert cycles per test before escalating to user review
5. **Escalation** - If 2 reverts occur for the same test, Forge pauses and reports:
   ```
   ESCALATION: Test [testId] has regressed 2x after fix attempts.
   Original failure: [description]
   Cascade failure: [description]
   Attempted fixes: [list]
   Recommendation: Manual review required.
   ```

### Agent Disagreement Resolution

When two agents disagree (e.g., Bug Fixer wants to change a file that Spec Verifier says shouldn't change):

1. **Quality Gate Enforcer acts as arbiter** - It evaluates both proposed states
2. **The change that results in more gates passing wins**
3. **Tie-breaking order:**
   - Fewer files changed (prefer minimal diff)
   - Higher confidence tier (prefer proven patterns)
   - Bug Fixer defers to Spec Verifier (specs are source of truth)

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
# forge.contexts.yaml - Project-specific bounded contexts and screens
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
    blocks: [orders, billing, users]
  payments:
    depends_on: [identity]
    blocks: [orders, subscriptions]
```

This separates the generic Forge engine from project-specific configuration, making Forge reusable across any codebase.

---

## QUICK REFERENCE CHECKLIST

Before running Forge:
- [ ] Plan approved via EnterPlanMode
- [ ] Backend built and running
- [ ] Health check passes
- [ ] Test data seeded via real API calls
- [ ] No internal mocking - external services only (see Mocking Policy)
- [ ] Gherkin specs exist for target context (or will be generated)
- [ ] All new screens/pages have test coverage
- [ ] Edge cases documented and tested

After Forge completes:
- [ ] Gate 1 (Functional): All tests pass
- [ ] Gate 2 (Behavioral): All targeted Gherkin scenarios satisfied
- [ ] Gate 3 (Coverage): >=85% overall, >=95% critical paths
- [ ] Gate 4 (Security): No hardcoded secrets, no injection vectors, no critical CVEs
- [ ] Gate 5 (Accessibility): WCAG AA compliance checked
- [ ] Gate 6 (Resilience): Offline/timeout/error states tested
- [ ] Gate 7 (Contract): API responses match expected schemas
- [ ] Confidence tiers updated for all applied fix patterns
- [ ] Defect predictions updated for next run
- [ ] All fixes committed with detailed messages
