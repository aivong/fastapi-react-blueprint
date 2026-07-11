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
