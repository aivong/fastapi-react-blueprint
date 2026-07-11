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

---

## Bug Triage Guide

*"I found this bug in production. What should have caught it?"*

Use these examples to diagnose which tier to invest in when a bug escapes.

---

### Tier 0 would have caught it

**Symptom**: `Cannot read property 'name' of undefined` in production.
**Root cause**: A function returns `string | undefined` but the caller assumes it always returns `string`. No null check.
**What to add**: Strict type checking with `strictNullChecks` enabled. The type checker would flag every unguarded access at build time. Zero runtime cost.

**Symptom**: Dead code path silently skipped — a feature flag check uses `=` instead of `==`.
**Root cause**: Assignment instead of comparison in a conditional.
**What to add**: A linter rule that flags assignments in conditionals. Caught before tests even run.

---

### Tier 1 would have caught it

**Symptom**: Users charged the wrong amount — discount applied as 15% instead of capping at $50.
**Root cause**: Business logic uses `price * discount` when it should be `Math.min(price * discount, maxDiscount)`.
**What to add**: Unit test with a price high enough that `price * discount > maxDiscount`. Mutation testing would have flagged this — removing the `Math.min` call would survive if no test checks the cap.

**Symptom**: Users born on Jan 1 are shown as one year younger than they are.
**Root cause**: Off-by-one in date comparison — `>` instead of `>=` on the birthday boundary.
**What to add**: Unit test for the exact boundary value (birthday = today). Mutation testing catches `>=` vs `>` swaps.

---

### Tier 2 would have caught it

**Symptom**: Frontend shows "undefined" where a user's display name should be.
**Root cause**: Backend renamed the field from `userName` to `displayName` in the API response. Frontend still reads `userName`.
**What to add**: Generate frontend types from the backend's API schema. The build fails the moment the field name changes — no runtime surprise.

**Symptom**: Frontend form submits successfully but backend returns 422 Unprocessable Entity.
**Root cause**: Backend added a new required field `phoneNumber` to the request body. Frontend never sends it.
**What to add**: Schema validation in CI — the generated types would show `phoneNumber` as required, and TypeScript would flag every call site that doesn't provide it.

---

### Tier 3 would have caught it

**Symptom**: Search works in development but returns no results in production.
**Root cause**: Query uses `ILIKE` (case-insensitive search) which works in PostgreSQL but was tested against an in-memory mock that didn't enforce SQL dialect.
**What to add**: Integration test against a real database container. The test would either pass with real PostgreSQL behavior or reveal the dialect mismatch.

**Symptom**: Third-party API calls succeed in tests but fail in production with 429 Too Many Requests.
**Root cause**: Tests mock the API with `unittest.mock`, which always returns 200. Production hits the rate limit.
**What to add**: A stateful fake that tracks call count and returns 429 after the limit. Tests the retry/backoff behavior of your orchestrator.

---

### Tier 4 would have caught it

**Symptom**: Login works when tested in isolation but fails when navigating from the pricing page.
**Root cause**: The pricing page sets a cookie that conflicts with the auth flow. No single unit or integration test covers this cross-page interaction.
**What to add**: E2E smoke test that navigates the full happy path: landing → pricing → login → dashboard.

**Symptom**: "Submit" button is present in the DOM but users can't click it.
**Root cause**: A CSS z-index issue — a transparent overlay sits on top of the button. All unit and integration tests pass because they don't render real CSS.
**What to add**: Visual regression test (screenshot comparison) or a Playwright test that actually clicks the button in a real browser.

---

### Quick Reference: Bug → Tier

| Bug you found in prod | Tier that catches it | Type of test to add |
|---|---|---|
| Type error, null reference, wrong argument type | **Tier 0** | Strict type checking |
| Wrong calculation, off-by-one, bad boundary | **Tier 1** | Unit test + mutation testing |
| Field renamed/added/removed in API | **Tier 2** | Schema-generated types in CI |
| Query works in mock but not real database | **Tier 3** | Integration test with container |
| External API rate limit, retry logic broken | **Tier 3** | Stateful fake |
| Cross-page interaction bug | **Tier 4** | E2E smoke test |
| Visual/CSS regression | **Tier 4** | Screenshot comparison (VRT) |

---

## Specialized Testing for Agentic & Orchestrator Applications

If your application acts as an agent orchestrator, manages dynamic task lifecycles, spawns child subprocesses/containers, or dynamically generates files/code artifacts (e.g., projects like Symphony or OpenHands), you face unique failure modes. Add these specialized sub-types to your testing strategy:

### Tier 1: Unit & Mutation Sub-types (No I/O)

#### Stale Cache Invalidation Testing
*   **When to use**: Your app dynamically generates code, scripts, or configuration files to the disk.
*   **Symptom**: Clean-slate CI/CD test runs pass, but developers/users experience stale caching regressions locally because previous outputs are not overwritten.
*   **Test Pattern**: In your unit test, proactively seed the output directory with a "stale garbage" file, run the generator, and assert that the stale garbage was completely overwritten or cleaned.

#### Resource Cleanup & Disk Leak Testing
*   **When to use**: Your app creates temporary files, workspaces, or session data directories.
*   **Symptom**: Disk space leaks over time on the server, especially when runs fail or throw exceptions mid-way, bypassing standard cleanup blocks.
*   **Test Pattern**: Force an exception/failure in the middle of the agent's work loop and assert that cleanup blocks (`try...finally`) still execute and delete all temporary files/directories.

#### Composition Root Testing
*   **When to use**: Your app dynamically resolves prompting templates, tool adapters, or LLM clients via dependency injection.
*   **Symptom**: Production boots up but silently resolves the wrong concrete adapter (e.g., using a mock or fallback model in prod).
*   **Test Pattern**: Write unit tests that instantiate the composition root/resolver and assert that it correctly resolves to the expected production types.

---

### Tier 3: Component Integration Sub-types (Ephemerals & Fakes)

#### Chaos & Fault Injection Testing (Advanced Fakes)
*   **When to use**: Your orchestrator manages multi-step execution flows depending on fragile third-party APIs (LLM endpoints, external workspaces, APIs).
*   **Symptom**: Upstream 502/504/timeout errors crash the agent loop midway or lead to infinite retry loops.
*   **Test Pattern**: Use stateful fakes to simulate catastrophic external API failures (e.g. return 504 Gateway Timeout) and assert your orchestrator degrades gracefully (retries with backoff, saves current state, returns a clean error).

#### Background Worker & Subprocess Observability Testing
*   **When to use**: Your orchestrator spawns background processes, workers, or containerized execution sandboxes.
*   **Symptom**: Child tasks crash silently with impossible-to-debug 0-byte log files (due to unflushed IO buffers) or log files are overwritten by concurrent test/task runs.
*   **Test Pattern**:
    *   **Isolate Logs**: Assert that every process/session log filename includes a UUID or unique task ID to prevent collisions.
    *   **Force Flushes**: Assert that the worker script explicitly calls flush/sync on log writers so crashes don't lose the buffer.
    *   **Never Swallow Stderr**: Avoid piping stderr to null; assert that raw stderr streams are redirected to diagnostic log files.

---

### Bug Triage: Orchestration/Agentic Examples

#### Symptom: A background task crashes silently, but its log file is empty (0 bytes).
*   **Root cause**: The child script crashed before its file buffer was flushed or closed.
*   **Tier to add**: **Tier 3 (Subprocess Observability)**. Ensure the script calls `flush()` immediately after critical writes, or use unbuffered output logging.

#### Symptom: The agent loop crashes when the LLM provider experiences a brief 502/504 gateway timeout.
*   **Root cause**: Missing retry/graceful degradation logic in the LLM connector.
*   **Tier to add**: **Tier 3 (Fault Injection)**. Use a fake LLM provider in tests to return a 504, and assert the loop retries or pauses rather than crashing.

#### Symptom: Workspace files from previous runs leak disk space.
*   **Root cause**: Cleanup code is skipped on unhandled exceptions.
*   **Tier to add**: **Tier 1 (Resource Cleanup)**. Write a test that triggers an exception mid-run and verify the cleanup block still runs.


