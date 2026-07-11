---
name: shift-left-testing-pyramid
version: "1.0.0"
description: "Tech-stack-agnostic 5-Tier Shift-Left Testing Pyramid. Use when designing a testing strategy, deciding what kind of test to write, or reviewing test coverage gaps across any stack."
tags:
  - testing
  - qa
  - strategy
  - pyramid
---

# 5-Tier Shift-Left Testing Pyramid

Shift validation left in the development cycle — catch bugs at the cheapest tier possible. Each tier is more expensive to run and slower to feedback than the one below it. Invest accordingly: heavy at the base, thin at the top.

```
            ┌───────────┐
            │  Tier 4   │  Automated E2E / System Tests
            │  (slow)   │  Minutes. Full-stack smoke tests.
            ├───────────┤
            │  Tier 3   │  Component Integration
            │           │  Seconds–minutes. Real services in containers.
            ├───────────┤
            │  Tier 2   │  Contract Tests
            │           │  Seconds. Schema validation, no runtime.
            ├───────────┤
            │  Tier 1   │  Unit & Mutation Tests
            │  (fast)   │  Milliseconds. Pure domain logic.
            ├───────────┤
            │  Tier 0   │  Static Analysis
            │  (instant)│  Pre-test. Type checkers & linters.
            └───────────┘
```

For implementation-level test patterns (factories, behavior-driven assertions, public-API testing), load the `testing` skill. For TDD workflow, load the `tdd` skill.

---

## Tier 0: Static Analysis (Pre-Test)

Run type checkers and linters **before** any test suite executes. These catch entire categories of bugs (type mismatches, unreachable code, unused imports) at zero runtime cost.

**Principles:**
* Type checking and linting are the first gate in CI — fail fast.
* Strict mode by default. Opt out per-line, never per-project.
* Treat linter warnings as errors in CI.

---

## Tier 1: Unit & Mutation Tests

Fast, isolated tests for core domain logic. No I/O, no network, no database.

**Principles:**
* Write tests using strict Test-Driven Development (load `tdd` skill).
* Test behavior through public APIs, not implementation details (load `testing` skill).
* Run mutation testing against domain code to prove tests catch injected bugs — coverage alone is not proof of effectiveness (load `mutation-testing` skill).
* Target: domain logic at >90% mutation score.

---

## Tier 2: Contract Tests (Schema Validation)

Validate that the frontend and backend agree on types — at build time, with zero runtime cost. This is why contract tests sit before integration tests in the pyramid: you don't need a running server to catch a broken API contract.

**Principles:**
* **Frontend-backend type safety**: Generate frontend types directly from the backend's API schema (OpenAPI, GraphQL SDL, Protobuf). A schema change that breaks the frontend fails the build, not production. Never maintain handwritten interface definitions that can drift.
* **Schema-first**: The API schema is the single source of truth. Both sides generate their types from it. Drift is impossible when there's only one definition.
* Shared schema files are treated as immutable source code — versioned, reviewed, never auto-generated from application state.
* Contract tests run in CI on every PR — they are fast (no runtime services needed) and should block merge on failure.

---

## Tier 3: Component Integration (Containers & Fakes)

Test infrastructure adapters against real backing services.

**Principles:**
* Use ephemeral containers (e.g., Testcontainers) to spin up real databases, message brokers, and caches. No shared test databases.
* **Virtualization tolerance**: When asserting against containerized services, design with generous timeouts to tolerate hypervisor and network-bridge latency. Flaky ≠ slow — a 2-second timeout that works on bare metal may need 10 seconds on nested virtualization.
* **Fakes over mocks**: For external APIs, build stateful fakes that maintain in-memory state and test the *behavior* of your orchestration code. Mocks that only assert `called_with()` prove nothing about correctness.
* Each integration test is idempotent — creates its own state, cleans up after itself, runs in any order.

---

## Tier 4: Automated E2E / System Tests

Full-stack smoke tests that prove the system works end-to-end. Keep these thin — most bugs should already be caught at lower tiers.

**Principles:**
* Write an E2E smoke test *immediately* after wiring up a new integration boundary (e.g., frontend-to-backend proxy, API gateway) to catch fundamental connectivity issues.
* Use browser automation for UI smoke tests. Query by accessible roles and text, not CSS selectors (load `webapp-testing` skill).
* **Visual Regression Testing (VRT)**: Screenshot comparison to assert visual styling stability. Use dynamic masks for non-deterministic content (timestamps, avatars).
* Programmatic E2E: For non-UI systems, execute the full workflow programmatically and assert outcomes mathematically (e.g., run generated code via subprocess, assert exit code and output).

---

## Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|---|---|---|
| **Ice cream cone** (most tests at E2E, few unit) | Slow feedback, flaky, expensive to maintain | Invert: heavy unit, thin E2E |
| **Mocking everything** at Tier 3 | Proves nothing about real integrations | Use containers + fakes |
| **No contract tests** | Schema drift discovered in staging/prod | Add schema validation to CI |
| **Skipping Tier 0** | Type errors caught at runtime instead of build | Strict type checking + linting first |
| **Coverage theater** | 100% line coverage with no assertions | Mutation testing proves assertion quality |
| **Shared test databases** | State bleeds between tests, flaky failures | Ephemeral containers per test/suite |

---

## Tier Selection Guide

When deciding what kind of test to write:

1. **Pure logic, no I/O?** → Tier 1 (unit test)
2. **Does it depend on an API schema?** → Tier 2 (contract test)
3. **Does it talk to a database, cache, or queue?** → Tier 3 (integration test with container)
4. **Does it need the full system running?** → Tier 4 (E2E smoke test)
5. **Always**: Tier 0 runs before everything else
