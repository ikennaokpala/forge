# Changelog

All notable changes to the Forge project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2026-02-20

### Added

- **12 topological governance sections (§1.1–§1.12)** establishing mathematical foundations for autonomous QE:
  - §1.1 Sheaf-Theoretic Consistency Model - global consistency via sheaf cohomology, H^1 ≠ 0 triggers commit rejection
  - §1.2 Sheaf Laplacian & Dirichlet Energy - system tension quantification with stability threshold E(S) ≤ 0.7
  - §1.3 Persistent Sheaf Laplacian - commit-history filtration with persistence barcodes for Stable/Flaky/Regressed classification
  - §1.4 Hallucination Gate - deterministic 3-phase pre-LLM verification (AST resolution, contract hash, internal mocking detection)
  - §1.5 Blake3 Cryptographic Witness Chain - append-only hash chain for gate verdicts in `forge-witnesses` namespace
  - §1.6 Algebraic Connectivity & Spectral Analysis - Fiedler value λ₂ monitoring with fragmentation thresholds
  - §1.7 Dynamic MinCut Isolation - quarantine protocol for anomalous agent output
  - §1.8 Hyperbolic Memory Architecture - Poincaré ball embeddings with HNSW readiness marker
  - §1.9 GF(3) Triadic Validation - Galois field phase transition governance with conservation law
  - §1.10 Narya-Proofs - counterfactual verification (necessity + sufficiency) for every fix
  - §1.11 Sublinear Coverage via Johnson-Lindenstrauss - random projection for O(log n) representative test selection
  - §1.12 WASM/Rust Execution Plane - six pure-function tasks specified for WASM compilation
- **8 inline agent enhancements** integrating topological foundations into agent prompts:
  - Bug Fixer: Driver-Observer Algebraic Connectivity (λ₂ monitoring)
  - Failure Analyzer: MaTTS - 3 parallel reasoning trajectories with self-contrast
  - Learning Optimizer: DISTILL phase - LoRA-style abstraction with EWC++ anti-forgetting
  - Gate Enforcer: BFT Consensus Model - ≥5/7 threshold with blocking gate VETO power
  - Execution Loop: SPARC Pipeline Mapping (8-phase → 5-stage)
  - LLM-as-Judge: Anti-Echo-Chamber Guarantee - ≥2 model architectures, P(both wrong) < 0.25
  - Confidence Tiers: Nash Equilibrium Property - break-even at P(success) ≥ 2/3
  - Quality Gates: Continuous Verification Daemon - incremental evaluation with Čech nerve structure
- **`forge-witnesses` memory namespace** for cryptographic witness chain and Narya-proof storage
- **Infrastructure readiness markers** for Blake3, HNSW, JL projections, and WASM/Rust runtime

### Changed

- **Concept coverage**: 83% → 100% across 29 evaluation dimensions from advanced multi-agent governance theory
- **Blog post** (`docs/blog/forge-v1-evolution.md`): updated with v1.2.0 topological governance narrative, 4 new sections, updated roadmap

### Validated

- All 29 concepts from "Advanced Topological Governance in Autonomous Software Engineering" now have formal specifications
- 12 topological sections integrate cleanly with existing 8-agent autonomous loop
- Infrastructure readiness markers defined for all specification-to-runtime transitions
- Every inline agent enhancement anchored to a specific topological specification

[1.2.0]: https://github.com/ikennaokpala/forge/compare/v1.1.0...v1.2.0

## [1.1.0] - 2026-02-20

### Added

- **External-only mocking policy** with Allowed/Forbidden lists and 3-point enforcement (Coverage Validator scan, Gate 4 block, Auto-Committer refusal)
- **Spec drift detection** with 3 types: static drift (code paths vs Gherkin steps), contract drift (API specs vs endpoint definitions), behavioral regression (50-result history with Stable/Flaky/Regressed classification)
- **LLM-as-Judge meta-evaluation** with 5-dimension rubric (Functional Completeness, Error Handling, Contract Alignment, Existence Verification, Test Quality). Auto-activates below Silver confidence, manual via `--meta-review`
- **Bug Fixer self-reflection gate** (Step 3.5) with 5 dimensions: Completeness, Error Handling, Edge Cases, Contract Adherence, Existence Verification
- **Fix strategy prioritization** - 4-tier priority order: PREFERRED (real implementation + in-memory DB), ACCEPTABLE (real + external-only mocks), LAST RESORT (contract-first), NEVER (mock internal services - 0% success rate evidence)
- **Property-based testing** - invariant extraction from Gherkin `Then` clauses and ADR `MUST` constraints, 1000+ random test cases per invariant with automatic shrinking. Frameworks: Dart (`check`), JavaScript/TS (`fast-check`), Python (`hypothesis`), Rust (`proptest`/`quickcheck`), Go (`rapid`)
- **Mutation testing** - operator flipping, constant changes, null check removal, condition swapping. Kill-rate targets: >=85% critical paths, >=70% overall codebase
- **Agent criticality & bottleneck detection** - weighted scoring formula `(0.30 × duration) + (0.30 × blocking_impact) + (0.20 × cost) + (0.20 × detection_rate)`, thresholds >0.8 Critical / 0.5–0.8 Moderate / <0.5 Healthy, 5 auto-recommendations (AddParallelism, UpgradeModel, DowngradeModel, ReorderExecution, CacheResults)
- **MODE-SPECIFIC BEHAVIOR section** - behavioral specs for `--verify-only`, `--drift-check`, `--meta-review`, `--mutation --critical-only` with agent spawn lists, phase execution rules, and JSON output format examples
- **Mandatory Plan Before Execute directive** - every Forge invocation must call `EnterPlanMode` before executing
- **8 new invocation modes**: `--drift-check`, `--regressions`, `--meta-review`, `--mutation`, `--mutation --critical-only`, `--predict`, `--chaos`, `--spec-gen`
- **`forge-criticality` memory namespace** for agent performance trend analysis across runs

### Changed

- **Mocking policy**: from absolute "no mocking" ban to nuanced external-only policy with Allowed/Forbidden lists, code examples, and enforcement mechanisms
- **Bug Fixer prompt**: added fix strategy prioritization (4-tier order with production evidence) and self-reflection gate (Step 3.5)
- **Gate 4 (Security)**: enhanced with internal mocking violation check - blocks commits containing forbidden mock patterns
- **Spec Verifier**: enhanced with drift analysis steps (static, contract, behavioral regression)
- **Autonomous loop**: Plan step added as mandatory entry point before execution

### Fixed

- **Table rendering**: escaped pipe characters in Rust `proptest` example (SKILL.md L1400) that broke markdown table formatting

### Validated

- End-to-end Forge run against production client project (20 bounded contexts, 717 backend tests, 726 Gherkin scenarios, 10 OpenAPI specs)
- 7 quality gates evaluated without errors
- All 8 agent definitions verified intact
- SKILL.md validated against advanced multi-agent governance theory (83% concept coverage across 29 dimensions)

### Issues Addressed

This release directly addresses 13 GitHub issues:

| Issue | Title |
|-------|-------|
| #1 | Spec Drift Detection |
| #2 | Behavioral Regression Alerts |
| #5 | Property-Based Testing for AI Code Quality |
| #7 | LLM-as-a-Judge for Automated Code Review |
| #10 | Self-Reflection Patterns for Pre-Submission Quality |
| #11 | Mutation Testing |
| #20 | Agent #3 Results: LLM-as-Judge Found 0.4% Test Coverage |
| #21 | Agent #6 Results: Self-Reflection Caught Non-Existent Widget |
| #22 | Autonomous QA Orchestra Results: 48 Issues in 16 Minutes |
| #24 | Policy Clarification: Mocking/Stubbing Guidelines |
| #25 | Learnings from Production Orchestra |
| #26 | 100% Success Rate + Cyclomatic Path Coverage |
| #27 | Agent Criticality Analysis & Bottleneck Detection |

## [1.0.0] - 2026-02-07

### Added

#### Core Skill (SKILL.md)

- **Autonomous QE Swarm**: 8 specialized agents spawned in parallel for continuous quality verification
  - Specification Verifier (Sonnet) - generates/validates Gherkin behavioral specs and ADRs
  - Test Runner (Haiku) - executes E2E test suites, parses results, maps failures to specs
  - Failure Analyzer (Sonnet) - root cause analysis, pattern matching, dependency impact assessment
  - Bug Fixer (Opus) - applies confidence-tiered fixes from first principles
  - Quality Gate Enforcer (Haiku) - evaluates all 7 gates, arbitrates agent disagreements
  - Accessibility Auditor (Sonnet) - WCAG AA audit: labels, contrast, targets, focus order
  - Auto-Committer (Haiku) - stages fixed files, creates detailed commits with gate statuses
  - Learning Optimizer (Sonnet) - updates confidence tiers, defect prediction, coverage metrics

- **7 Quality Gates**: Evaluated after every fix cycle, all must pass before commit
  - Gate 1 - Functional (100% pass rate, blocking)
  - Gate 2 - Behavioral (100% Gherkin scenario coverage, blocking)
  - Gate 3 - Coverage (>=85% overall, >=95% critical paths, blocking for critical)
  - Gate 4 - Security (0 critical/high violations including SAST checks, blocking)
  - Gate 5 - Accessibility (WCAG AA compliance, warning)
  - Gate 6 - Resilience (offline/timeout/error handling, warning)
  - Gate 7 - Contract (0 API schema mismatches, blocking)

- **Model Routing**: Cost-optimized routing of agents to appropriate model tiers
  - Opus for high-reasoning tasks (Bug Fixer)
  - Sonnet for moderate-reasoning tasks (Spec Verifier, Failure Analyzer, A11y Auditor, Learning Optimizer)
  - Haiku for low-reasoning tasks (Test Runner, Gate Enforcer, Auto-Committer)
  - Configurable via `forge.config.yaml` model_routing overrides
  - Estimated ~60% token cost reduction vs running all agents on highest tier

- **Confidence-Tiered Fix Patterns**: Self-evolving pattern database
  - Platinum (>=0.95) - auto-apply immediately without review
  - Gold (>=0.85) - auto-apply with commit message flag
  - Silver (>=0.75) - suggest to Bug Fixer, don't auto-apply
  - Bronze (>=0.70) - store for learning only
  - Expired (<0.70) - demoted, needs revalidation
  - Success: confidence += 0.05, Failure: confidence -= 0.10

- **Behavioral Specification & Architecture Records (Phase 1)**:
  - Gherkin spec generation from implementation analysis
  - Spec-to-test mapping with zero-gap coverage
  - Agent-optimized ADR generation with MUST/MUST NOT/Verification/Dependencies template
  - ADR storage in `docs/decisions/` or project-configured directory

- **Contract & Dependency Validation (Phase 2)**:
  - API response schema validation against expected DTOs
  - Shared types validation across bounded context boundaries
  - Cross-cutting foundation validation (auth, error format, logging, pagination consistency)
  - Dependency graph with cascade re-testing when fixes touch upstream contexts

- **Structured Agent Prompts**: All 8 agents include
  - CONSTRAINTS section - explicit rules about what NOT to do
  - ACCEPTANCE section - machine-checkable success criteria

- **Autonomous Execution Loop**: Specify → Test → Analyze → Fix → Audit → Gate → Commit → Learn → Repeat
  - Max 10 iterations per run
  - Gate failures categorized for targeted re-runs (not full re-run)

- **Real-Time Progress Reporting**:
  - Structured JSON events from all agents written to `.forge/progress.jsonl`
  - Integration with AG-UI streaming protocol when Agentic QE is available

- **Defect Prediction**: Proactive failure prediction based on
  - Files changed since last green run
  - Historical failure rates per bounded context
  - Fix pattern freshness (recently applied fixes regress more)
  - Complexity metrics and dependency chain length
  - Tests executed in descending probability order for faster convergence

- **Exhaustive Edge Case Testing**: Comprehensive test patterns for
  - Interaction states (single, rapid, long press, disabled)
  - Input field states (empty, focus, valid, invalid, max length, paste, clear)
  - Async operation states (loading, success, error, timeout)
  - Navigation edge cases (back, deep link, invalid deep link)
  - Scroll edge cases (overscroll, hidden content, keyboard)
  - Network edge cases (offline, slow, reconnect, 500/401/403/404)
  - Chaos testing (timeout injection, partial response, rate limiting, concurrent mutations, session expiry)

- **Visual Regression Testing**:
  - Before/after screenshot capture and pixel-by-pixel comparison
  - Configurable threshold (default 0.1% diff tolerance)
  - Platform-specific capture (Playwright, Cypress, Flutter golden files)
  - Flagged as Gate 5 warnings

- **Rollback & Conflict Resolution**:
  - Rollback to last known good commit with pattern demotion
  - Fix Conflict Protocol - halt, re-analyze, categorize, revert limit (max 2 cycles)
  - Agent Disagreement Resolution - Gate Enforcer as arbiter, more-gates-passing wins
  - Escalation to user review after 2 failed revert cycles

- **Optional Agentic QE MCP Integration**:
  - Auto-detection via `claude mcp list`
  - ReasoningBank for vector-indexed pattern storage (replaces claude-flow memory)
  - Delegation to specialized AQE domains: defect-intelligence, security-compliance, visual-accessibility, contract-testing, learning-optimization, requirements-validation
  - AG-UI protocol for real-time progress streaming
  - All features additive - Forge works identically without AQE

- **Architecture Adaptability**: Supports monoliths, modular monoliths, microservices, monorepos, mobile+backend, full-stack monoliths
  - Auto-discovers backend/frontend technology, test framework, project structure, API protocol, build system
  - Optional `forge.config.yaml` for explicit configuration
  - Optional `forge.contexts.yaml` for bounded context definitions

- **Defensive Test Patterns**: Framework-specific safe interaction helpers
  - Flutter: safeTap, safeEnterText, visualDelay, scrollUntilVisible, waitForApiResponse
  - Cypress/Playwright: safeClick, waitForApi

- **Common Fix Patterns**: Pre-loaded patterns for common failure types
  - Element Not Found (Platinum, 0.97 confidence)
  - Timeout (Gold, 0.89 confidence)
  - Assertion Failed (Silver, 0.78 confidence)
  - API Response Mismatch (Gold, 0.86 confidence)

- **Invocation Modes**: 14 CLI modes
  - `--autonomous --all` / `--autonomous --context [name]`
  - `--verify-only` / `--fix-only`
  - `--learn` / `--predict`
  - `--add-coverage --screens [names]`
  - `--spec-gen --context [name]` / `--spec-gen --all`
  - `--gates-only` / `--chaos`

- **No Mocking Rule**: Absolute rule - all tests run against real backend API, no mock frameworks allowed

- **Memory Namespaces**: 8 namespaces for persistent state
  - forge-patterns, forge-results, forge-state, forge-commits, forge-screens, forge-specs, forge-contracts, forge-predictions

#### Documentation (README.md)

- Comprehensive project documentation with:
  - Project overview and philosophy (Build/Verify/Heal pillars, "DONE DONE" concept)
  - Quick start guide
  - Full invocation modes table (14 CLI modes)
  - Architecture diagram (autonomous loop visualization)
  - Quality gates reference table
  - Agent roles table with model tiers
  - Configuration examples (forge.config.yaml, forge.contexts.yaml)
  - Agentic QE integration capability matrix
  - Links to all 4 reference sources (CBV article, BWQ article, V3 QE Skill, Agentic QE)
  - MIT license

#### Project Setup

- `.gitignore` - excludes node_modules, .DS_Store, logs, .env, .dev.vars
- `LICENSE` - MIT license

### Reference Sources

This release was validated against and incorporates concepts from:

1. **Continuous Behavioral Verification** (Ikenna Okpala) - "DONE DONE" concept, Gherkin behavioral contracts, artifact layer, agent-optimized ADRs
2. **Build with Quality Skill** (Mondweep Chakravorty) - DDD+ADR+TDD pillars, hierarchical-mesh agent topology, quality gates during development
3. **V3 QE Skill** (mondweep/vibe-cast) - Confidence tiers, 7 quality gates, TinyDancer model routing, coverage thresholds
4. **Agentic QE** (proffesor-for-testing/agentic-qe) - ReasoningBank, 51 specialized agents, AG-UI protocol, BrowserSwarmCoordinator

[1.1.0]: https://github.com/ikennaokpala/forge/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/ikennaokpala/forge/releases/tag/v1.0.0
