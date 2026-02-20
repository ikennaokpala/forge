# What 13 Production Issues Taught Us About Autonomous Quality Engineering

*February 2026*

---

On February 12, 2026, we launched five autonomous QA agents against a production mobile application. Two agents completed within 16 minutes. They found 48 issues — including a widget that doesn't exist in Flutter's SDK, silently crashing three core features. Another agent reviewed 250+ UI screen files and discovered test coverage was at 0.4%.

These results didn't just validate the approach. They exposed specific gaps in Forge's v1.0.0 design that no amount of theoretical planning would have surfaced. This post documents what those 13 production issues taught us and how they shaped Forge v1.1.0.

## The Problem: Why Autonomous QE Needs More Than "Run Tests and Fix Bugs"

Forge v1.0.0 shipped with eight specialized agents, seven quality gates, confidence-tiered fix patterns, and an autonomous execution loop. On paper, it covered the full quality lifecycle — specify, test, analyze, fix, gate, commit, learn.

In production, three categories of problems emerged:

**1. The mocking trap.** Forge v1.0.0 had an absolute rule: "This skill NEVER uses mocking or stubbing of the backend API." In practice, agents needed to mock *something* — Stripe payment processing, Firebase push notifications, Twilio SMS. The blanket ban forced agents into impossible situations: either violate the rule or skip testing external integrations entirely. Five pull requests failed because agents mocked internal services instead. Five succeeded because they used real implementations. The policy needed nuance, not absolutism.

**2. Shallow fixes that compile but don't work.** The Bug Fixer agent could generate code that passed compilation and even basic tests. But it had no mechanism to ask itself "does this widget actually exist?" or "did I leave a TODO stub?" Agent #6 discovered that `RadioGroup<T>` — used in three screens — isn't a real Flutter widget. The code compiled. Static analysis passed. No tests caught it. Only a self-reflective "what could go wrong?" pass found it.

**3. Spec drift going undetected.** Between releases, implementation changes accumulated without corresponding Gherkin spec updates. API response fields changed names. New endpoints appeared without scenarios. Old scenarios referenced removed code paths. The specs said one thing; the code did another. Without active drift detection, the gap between "specified behavior" and "actual behavior" widened silently.

## Production Evidence: The Numbers That Changed Everything

### The Orchestra Experiment (Issue #22)

Five agents ran in parallel against a production client project:

| Agent | Approach | Runtime | Findings |
|-------|----------|---------|----------|
| Agent #3 | LLM-as-Judge meta-evaluation | 7 min 20 sec | 0.4% test coverage, 83 TODOs, missing error handling |
| Agent #6 | Self-reflection ("what could go wrong?") | 8 min 41 sec | 41 issues across 28 screens, RadioGroup<T> crash |
| Agent #1 | Property-based testing | In progress | Resource allocation invariants |
| Agent #2 | BDD user journeys | In progress | 5 critical end-to-end workflows |
| Agent #11 | Runtime verification | In progress | Telemetry and invariant checking |

Two agents, 16 minutes, 48 issues. The completed agents found fundamentally different categories of problems — Agent #3 found systemic gaps (coverage, technical debt), while Agent #6 found a specific production-breaking bug that every other quality check missed.

This wasn't additive. It was multiplicative. Each perspective caught issues invisible to the others.

### The 0.4% Coverage Discovery (Issue #20)

Agent #3 used Claude Opus as a meta-evaluator across 250+ UI screen files. In 7 minutes and 20 seconds, it identified:

- **1 test file** covering **250+ screens** — 0.4% test coverage
- **83 TODOs/FIXMEs** scattered across the codebase — incomplete features shipped as complete
- Missing error handling on async operations — silent failures in production
- No API contract verification — frontend and backend could diverge without detection

The agent didn't just report these findings. It created GitHub issues with detailed analysis and submitted a pull request with 16 new tests (14 of which passed immediately). Manual review of the same codebase would have taken an estimated 2-3 days. The LLM-as-Judge completed it in 7 minutes with higher consistency and exhaustive coverage.

### The RadioGroup<T> Crash (Issue #21)

Agent #6 analyzed 28 UI screens asking "what could go wrong?" for each one. On `request_screen.dart`, line 247, it found code using `RadioGroup<T>` — a widget that does not exist in Flutter's SDK.

Three core features were completely broken:
- **Request rejection** — app crashes when user taps "Decline"
- **Form creation** — crashes during option selection
- **Payment processing** — crashes when selecting payment account

How did this go undetected?
- The code compiled successfully (Dart allows non-existent generic types)
- No static analysis errors flagged it
- Pull request reviews missed it
- Test coverage was 0.4% — no test exercised these paths

The fix was straightforward: replace `RadioGroup<T>` with standard Flutter `RadioListTile<String>` widgets. The agent implemented the fix, submitted a PR, and the three features worked again.

**Compilation does not equal correctness.** This single finding justified the entire self-reflection approach.

## Eight Changes Forged in Production

Each change below traces directly to one or more of the 13 closed issues, with production evidence backing the design decision.

### 1. External-Only Mocking Policy (Issues #24, #25, #26)

**What changed:** The absolute "never mock anything" rule was replaced with a nuanced external-only mocking policy.

**The evidence:** From the production orchestra run, we observed a perfect split:
- **5/5 PRs that mocked internal services failed** (100% failure rate)
- **5/5 PRs that used real implementations succeeded** (100% success rate)

The problem wasn't mocking itself — it was mocking *internal* code. When agents mocked `OrderService` or `PaymentRepository`, they created tests that passed against fakes but broke against real implementations. When agents mocked *Stripe* or *Firebase* (external services outside the system boundary), the tests remained valid because the mock accurately represented the external contract.

**The implementation:** Forge v1.1.0 specifies:

- **Allowed mocks:** Stripe, Firebase, Twilio, SendGrid, S3/R2, OAuth providers — services outside your system boundary
- **Forbidden mocks:** Your own services, models, repositories, controllers, middleware — internal code
- **3-point enforcement:** Coverage Validator scans for violations, Gate 4 blocks internal mock usage, Auto-Committer refuses commits containing forbidden mocks

This is the difference between policy-as-prohibition and policy-as-engineering-constraint.

### 2. Self-Reflection Gate (Issues #10, #21)

**What changed:** Bug Fixer gained a new Step 3.5 — a self-reflection gate that executes before any fix is stored or committed.

**The evidence:** Agent #6's discovery of `RadioGroup<T>` proved that a single "what could go wrong?" pass catches bugs invisible to compilation, static analysis, and code review. The self-reflection process took seconds per screen but found a production-breaking bug across three features.

**The implementation:** Before storing a fix, the Bug Fixer asks "what could go wrong?" across five dimensions:

1. **Completeness** — Are there TODOs, stubs, or placeholder implementations?
2. **Error handling** — Does every async operation have try-catch? Are loading states managed?
3. **Edge cases** — What happens with empty input, null values, concurrent access?
4. **Contract adherence** — Do field names match the Gherkin spec exactly?
5. **Existence verification** — Does every widget, class, function, and import actually exist in the SDK?

Dimension 5 exists specifically because of the RadioGroup<T> incident. If any dimension fails, the fix loops back for revision before it reaches Gate evaluation.

### 3. LLM-as-Judge Meta-Evaluation (Issues #7, #20)

**What changed:** A second-model review evaluates Bug Fixer output using a structured 5-dimension rubric.

**The evidence:** Agent #3 reviewed 250+ files in 7 minutes 20 seconds and found systemic issues that individual test runs would never surface — 0.4% coverage, 83 TODOs, missing error handling patterns. The LLM-as-Judge approach provides a fundamentally different evaluation perspective: it reads and reasons about code holistically rather than executing individual test cases.

**The implementation:** The meta-evaluation scores fixes across five rubric dimensions:

1. **Functional completeness** — No TODOs, no stubs, all paths implemented
2. **Error handling** — Try-catch on async operations, user-facing error messages
3. **Contract alignment** — Response fields and status codes match Gherkin expectations
4. **Existence verification** — All imports, references, and SDK types are valid
5. **Test quality** — Happy path, error path, and edge case coverage

The evaluation activates automatically when fix confidence falls below Silver (<0.75), and can be triggered manually via `--meta-review` for any context. This creates a two-stage quality check: the Bug Fixer's self-reflection catches immediate issues, then the LLM-as-Judge catches systemic patterns.

### 4. Spec Drift Detection (Issues #1, #2)

**What changed:** Forge now actively detects when Gherkin specs and implementation diverge, instead of waiting for test failures to surface the gap.

**The evidence:** Issue #1 identified the core problem: when code behavior changes but specs aren't updated (or vice versa), the disconnect goes unnoticed until something breaks in production. Issue #2 extended this to behavioral regressions — scenarios that passed yesterday but fail today.

**The implementation:** Three types of drift detection:

1. **Static drift** — Parse Gherkin Given/When/Then steps and verify matching code paths exist. Flag specs referencing removed code, or code paths with no matching scenario.
2. **Contract drift** — Compare API specs defined in Gherkin against actual endpoint definitions (OpenAPI specs or route declarations). Catch field renames, status code changes, and schema mismatches.
3. **Behavioral regression** — Store the last 50 results per scenario. Classify each as Stable (10+ consecutive passes), Flaky (alternating pass/fail), or Regressed (was stable, now failing). Correlate regressions with specific commits.

A dedicated `--drift-check` invocation mode runs only the Spec Verifier agent to produce a drift report without executing tests or applying fixes. Severity levels (BLOCKING, WARNING, INFO) let teams integrate drift detection into CI pipelines.

### 5. Fix Strategy Prioritization (Issues #25, #26)

**What changed:** The Bug Fixer now follows an explicit 4-tier priority order when generating fixes, instead of defaulting to generic "implement from first principles" guidance.

**The evidence:** From the production orchestra's merged PRs, a clear pattern emerged:

- **PR #460** (Vehicle Selection): Real implementation + in-memory state → approved
- **PR #459** (Error Messages): Utility pattern + extensive testing → approved
- **PR #465** (API Contracts): Contract-first documentation → merged

Every successful PR used real implementations. Every failed PR mocked internal services.

**The implementation:** The Bug Fixer evaluates fix strategies in this order:

1. **PREFERRED:** Real implementation with in-memory database — exercise the full internal stack
2. **ACCEPTABLE:** Real implementation with external-only mocks — mock Stripe, never OrderService
3. **LAST RESORT:** Contract-first approach — define the API contract, implement against it
4. **NEVER:** Mock internal services — 0% historical success rate

This isn't a preference — it's an evidence-backed ranking. The "NEVER" tier exists because the data showed a 0% success rate for internal mocking across all observed PRs.

### 6. Property-Based and Mutation Testing (Issues #5, #11)

**What changed:** Two complementary testing approaches were added to catch categories of bugs that example-based testing misses.

**The evidence:** Property-based testing and mutation testing address different failure modes:
- Example-based tests verify specific inputs produce expected outputs. They miss edge cases the developer didn't think of.
- Property-based tests define invariants that must hold for *all* inputs. They discover edge cases automatically through random generation.
- Mutation testing validates that tests actually catch bugs by injecting deliberate faults and checking that tests fail.

**The implementation:**

**Property-based testing** extracts invariants from two sources:
- Gherkin `Then` clauses (e.g., "Then the balance should be non-negative" → `balance(n) >= 0` for all n)
- ADR `MUST` constraints (e.g., "MUST maintain capacity invariant" → `allocated <= total` for all allocations)

Forge generates 1000+ random test cases per invariant with automatic shrinking to find minimal counterexamples. Framework support covers Dart (`check`), JavaScript/TypeScript (`fast-check`), Python (`hypothesis`), Rust (`proptest`/`quickcheck`), and Go (`rapid`).

**Mutation testing** injects four types of mutations into critical code paths:
- Operator flipping (`==` → `!=`, `>` → `>=`)
- Constant changes (boundary values, off-by-one)
- Null check removal
- Condition swapping

Kill-rate targets: >=85% for critical paths (authentication, payments, state machines), >=70% for overall codebase. Surviving mutants generate specific test gap recommendations with file, line, mutation description, and suggested test.

### 7. Agent Criticality Scoring (Issue #27)

**What changed:** Forge now scores each agent's criticality using a weighted formula and automatically recommends pipeline optimizations.

**The evidence:** After the orchestra experiment, we couldn't answer basic questions: Which agent was the bottleneck? Which gate rarely fails? Where would parallelization help most? Issue #27 proposed criticality analysis inspired by MinCut algorithms for dependency graphs, adapted to Forge's 8-agent scale.

**The implementation:** Each agent receives a criticality score from 0.0 to 1.0:

```
criticality = (0.30 × duration_ratio) + (0.30 × blocking_impact)
            + (0.20 × cost_ratio) + (0.20 × detection_rate)
```

- **duration_ratio** — Agent runtime relative to total pipeline duration
- **blocking_impact** — How many downstream agents are blocked while this agent runs
- **cost_ratio** — Token cost relative to total pipeline cost
- **detection_rate** — Ratio of issues found to total issues found across all agents

Thresholds: >0.8 Critical, 0.5–0.8 Moderate, <0.5 Healthy.

When an agent exceeds the Critical threshold, Forge automatically recommends one of five optimizations: AddParallelism, UpgradeModel, DowngradeModel, ReorderExecution, or CacheResults. These recommendations are stored in the `forge-criticality` memory namespace for trend analysis across runs.

### 8. Mode-Specific Behavioral Specs (Current Branch)

**What changed:** Each invocation mode now has a precise behavioral specification defining which agents spawn, which phases execute, and what output format is produced.

**The evidence:** Forge v1.0.0 defined 14 invocation modes in a table, but the exact behavior of modes like `--verify-only` or `--drift-check` was ambiguous. Does `--verify-only` run the Failure Analyzer? Does `--drift-check` evaluate quality gates? Without explicit behavioral specs, agent implementations could diverge from intent.

**The implementation:** Four modes received full behavioral specifications:

- **`--verify-only`** — Spawns Spec Verifier, Test Runner, Gate Enforcer only. Skips Bug Fixer, Auto-Committer, Learning Optimizer. Evaluates Gates 1, 2, and 7. No files modified.
- **`--drift-check`** — Spawns Spec Verifier only. Executes all three drift detection types (static, contract, behavioral regression). No tests run, no files modified.
- **`--meta-review`** — Forces LLM-as-Judge evaluation regardless of confidence tier. Can combine with `--autonomous` or run standalone against existing code.
- **`--mutation --critical-only`** — Scopes mutation testing to authentication, payment, and core state machine modules. Applies >=85% kill-rate threshold (vs >=70% for full `--mutation`).

Each specification includes JSON output format examples, making behavior machine-verifiable.

## The XP Observer Pattern: Continuous Validation at Every Stage

The eight changes above aren't independent features bolted onto the side of Forge. They form a multi-dimensional validation pattern — an observer that evaluates quality from multiple perspectives at every stage of the autonomous loop.

Consider what happens when the Bug Fixer generates a fix:

1. **Self-reflection gate** asks "what could go wrong?" across 5 dimensions
2. **LLM-as-Judge** evaluates the fix against a 5-dimension rubric
3. **Property-based tests** verify invariants hold for thousands of random inputs
4. **Mutation tests** confirm that tests actually catch bugs in the fix
5. **Drift detection** ensures the fix doesn't create spec-implementation divergence
6. **Mocking policy enforcement** verifies no internal services were mocked
7. **Criticality scoring** tracks whether this fix stage is becoming a bottleneck

No single check catches everything. The RadioGroup<T> bug survived compilation, static analysis, and code review — but self-reflection caught it. The 0.4% coverage gap was invisible to individual test runs — but the LLM-as-Judge found it. Internal mocking violations passed individual tests — but the enforcement gate blocked them.

The value is in the combination. Each observer catches what others miss. And because they run continuously inside the autonomous loop — not as a post-hoc review — issues are caught at the point of creation, not after deployment.

This is the core insight from the production evidence: quality engineering isn't a single gate. It's an ongoing observation from multiple angles, where each angle reveals blind spots in the others.

## Validation at Scale

Before releasing v1.1.0, the updated SKILL.md was validated against a production client project:

| Dimension | Scale |
|-----------|-------|
| Bounded contexts | 20 |
| Backend integration tests | 717 |
| Gherkin scenarios | 726 |
| OpenAPI specifications | 10 |
| Quality gates evaluated | 7 |
| Agent definitions verified | 8 |

All seven quality gates evaluated without errors. All eight agent definitions verified intact. The validation confirmed that the additions integrate cleanly with the existing autonomous loop rather than creating conflicts or regressions.

Additionally, the SKILL.md changes were validated against advanced multi-agent governance theory, achieving 83% concept coverage across 29 evaluation dimensions — strongest in orchestration/routing (4/6 aligned) and emergent dynamics anti-drift (4/4 aligned).

## What's Next

The 13 issues addressed in v1.1.0 came from a single production orchestra run. They represent the first iteration of a feedback loop: deploy agents → observe results → extract patterns → evolve the skill → deploy again.

Several areas are already visible for the next iteration:

- **Dynamic model routing based on criticality scores.** If the Bug Fixer consistently scores >0.8 criticality, automatically upgrade its model tier. If the Gate Enforcer consistently scores <0.3, downgrade to save tokens.
- **Cross-context regression detection.** When a fix in the `payments` context affects the `orders` context, Forge should detect the cascade before it reaches production.
- **Confidence tier evolution from mutation results.** If mutation testing reveals that a Platinum-confidence fix pattern has a 60% kill rate, it should be automatically demoted.
- **Real-time agent coordination.** The current sequential loop (specify → test → analyze → fix → gate → commit → learn) could be parallelized based on the dependency graph from criticality analysis.

The v1.1.0 changes came from production evidence, not speculation. The next evolution will come from the same source — observing what happens when these eight improvements run against real codebases and extracting the patterns that emerge.

---

*Forge v1.1.0 addresses issues #1, #2, #5, #7, #10, #11, #20, #21, #22, #24, #25, #26, and #27. All changes are grounded in production evidence from autonomous orchestra runs against client projects.*
