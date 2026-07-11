---
name: fastapi-react-testing
version: "2.0.0"
description: "FastAPI & React tool mappings for the 5-Tier Shift-Left Testing Pyramid. Stack-specific overlay — load shift-left-testing-pyramid for the generic strategy."
tags:
  - testing
  - qa
  - testcontainers
  - playwright
  - fakes
dependencies:
  - shift-left-testing-pyramid
  - front-end-testing
  - webapp-testing
---

# FastAPI & React Testing — Tool Mappings

This skill maps each tier of the [shift-left-testing-pyramid](../shift-left-testing-pyramid/SKILL.md) to specific FastAPI/React tooling. Load the pyramid skill first for the generic strategy and principles.

---

## Tier 0: Static Analysis → Astral Tooling

* **Type checking**: `ty` (Rust-based, extremely fast). Run before tests in CI.
* **Linting & formatting**: `ruff` — catches logical flaws, enforces style. Treat warnings as errors.
* **Frontend**: TypeScript strict mode (`noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`). ESLint with recommended configs.

---

## Tier 1: Unit & Mutation Tests

* **Backend**: `pytest` for domain logic. Follow TDD (load `tdd` skill). Run `mutmut` against domain modules to prove test effectiveness.
* **Frontend**: Vitest Browser Mode (preferred) or DOM Testing Library for behavior-driven UI tests (load `front-end-testing` skill).

---

## Tier 2: Contract Tests → Schema Validation

* **OpenAPI**: Use `openapi-typescript` to generate frontend types from FastAPI's `/openapi.json`. Schema drift is caught at compile time.
* **GraphQL** (if applicable): Extract queries into shared `.graphql` files (treated as immutable source code, never application state). Use `@graphql-codegen/cli` to validate against the live production schema during CI.

---

## Tier 3: Component Integration → Testcontainers & Fakes

* **Databases**: `testcontainers-python` to spin up ephemeral PostgreSQL instances. Zero shared state between tests.
* **Virtualization tolerance**: When writing async polling loops or network assertions against containerized services, use generous timeouts to tolerate hypervisor/network-bridge latency.
* **External API fakes**: Build fakes with `respx` — maintain in-memory state dictionaries, test orchestrator *behavior*. No brittle `assert_called_with()` mocks.

---

## Tier 4: Automated E2E → Playwright

* **Smoke tests**: Write an E2E smoke test immediately after wiring up the frontend-to-backend proxy to catch Docker networking issues.
* **Browser automation**: Playwright with accessible selectors (load `webapp-testing` skill).
* **Visual Regression Testing**: `.toHaveScreenshot()` with dynamic masks for timestamps, avatars, and other non-deterministic content.
* **Programmatic E2E**: For backend-only flows, execute the full orchestrator loop and mathematically assert outcomes (e.g., run generated code via `subprocess`, assert exit code and output).

---

## Bug Triage: FastAPI/React Examples

See the [shift-left-testing-pyramid](../shift-left-testing-pyramid/SKILL.md) Bug Triage Guide for generic scenarios. Below are the same patterns with exact tools and fixes for this stack.

---

### Tier 0: `ty` / `ruff` / TypeScript would have caught it

**Bug**: FastAPI endpoint returns `None` for a field typed as `str`. Frontend renders "undefined".
**Fix**: `ty` in strict mode flags `str` vs `str | None` mismatch in the Pydantic model. The bug never reaches a test.

**Bug**: React component reads `user.email` but the prop type allows `undefined`. Crashes in production.
**Fix**: TypeScript `strictNullChecks` flags the unguarded access. Add a null check or make the prop required.

---

### Tier 1: `pytest` + `mutmut` / Vitest would have caught it

**Bug**: Subscription billing charges annual users monthly after a pricing change.
**Fix**: `pytest` unit test for `calculate_billing(plan="annual")` with a price that differs from the monthly rate. `mutmut` would flag this — swapping `annual_price` for `monthly_price` survives if no test checks the distinction.

**Bug**: React form validation accepts empty strings as valid email addresses.
**Fix**: Vitest Browser Mode test: `fill(emailInput, '')`, `click(submitButton)`, `expect.element(errorMessage).toBeVisible()`.

---

### Tier 2: `openapi-typescript` / `@graphql-codegen/cli` would have caught it

**Bug**: Backend renames `created_at` to `createdAt` in a FastAPI response model. Frontend breaks.
**Fix**: `openapi-typescript` regenerates types from `/openapi.json`. TypeScript immediately flags every `created_at` reference in the frontend as a compile error. CI blocks the merge.

**Bug**: Backend adds a required `organization_id` field to a POST endpoint. Frontend form doesn't send it, gets 422.
**Fix**: Generated types show `organization_id` as required. TypeScript flags every `fetch`/`axios` call that omits it.

---

### Tier 3: `testcontainers-python` / `respx` would have caught it

**Bug**: Full-text search works in development (SQLite) but returns nothing in production (PostgreSQL).
**Fix**: `testcontainers-python` spins up a real PostgreSQL instance. The integration test runs the actual query and reveals that SQLite's `LIKE` is case-insensitive but PostgreSQL's isn't — use `ILIKE`.

**Bug**: External payment API returns 429 in production. Your code retries infinitely, causing a cascade failure.
**Fix**: `respx` fake that returns 429 after 3 calls. Test asserts your orchestrator backs off exponentially and eventually raises a clear error instead of retrying forever.

---

### Tier 4: Playwright would have caught it

**Bug**: Login works in all tests but fails in production when the user navigates from the `/pricing` page first.
**Fix**: Playwright E2E test: `page.goto('/pricing')` → click "Sign up" → complete login flow → assert dashboard loads. Catches the cookie/redirect conflict.

**Bug**: "Save" button is behind a sticky footer on mobile viewports. Users can't tap it.
**Fix**: `.toHaveScreenshot()` with a mobile viewport (375×667). VRT diff shows the button is obscured. Or: `page.getByRole('button', { name: /save/i }).click()` throws because Playwright can't click an element covered by another element.

