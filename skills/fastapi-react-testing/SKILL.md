---
name: fastapi-react-testing
version: "1.0.0"
description: "Guidelines and requirements for the 5-Tier Shift-Left Testing Pyramid in Python/React applications."
tags:
  - testing
  - qa
  - testcontainers
  - playwright
  - fakes
dependencies:
  - karpathy-guidelines
  - front-end-testing
  - webapp-testing
---

# FastAPI & React Shift-Left Testing Strategy

Use this skill when designing, writing, or reviewing tests for a full-stack Python/React application. We structure testing around a 5-Tier "Shift Left" pyramid to eliminate manual verification and guarantee correctness.

## 1. The 5-Tier "Shift Left" Pyramid

We aggressively shift validation left in the development cycle to catch bugs early.

### Tier 0: Static Analysis (Pre-Test)
* Use Astral's `ty` to catch type mismatches and `ruff` to catch logical flaws before any test suite is executed.

### Tier 1: Unit & Mutation Tests
* Write fast, isolated unit tests for core Domain logic using strict Test-Driven Development (TDD).
* **Frontend**: Use Vitest Browser Mode (preferred) or DOM Testing Library to write behavior-driven UI tests.
* **Backend**: Run Mutation Testing (`mutmut`) against the backend domain to mathematically prove that the test suite catches injected bugs (asserting that mutations are killed).

### Tier 2: Contract Tests (Schema Validation)
* Never guess if an external API changed.
* For GraphQL APIs, extract queries into shared `.graphql` files (treated as immutable source code, never Application State) and use the frontend's `@graphql-codegen/cli` to validate them against the live production schema during CI.

### Tier 3: Component Integration (Testcontainers & Fakes)
* Test infrastructure adapters using real backing services.
* Use `testcontainers-python` to spin up ephemeral PostgreSQL databases.
* **Virtualization Tolerance**: When writing asynchronous polling loops or network assertions against containerized services, always design with generous timeouts to tolerate underlying hypervisor or network-bridge latency.
* **Fakes over Mocks**: For external APIs, **build Fakes instead of Mocks** (using `respx`). Fakes maintain in-memory state dictionaries and test the *behavior* of the orchestrator, unlike brittle mocks that only assert `called_with()`.

### Tier 4: Automated E2E System Tests
* Programmatically execute the entire orchestrator loop and mathematically assert the outcomes (e.g., executing the agent's generated code via `subprocess` to prove it runs) to completely eliminate manual UI verification.
* Use Playwright patterns for robust, non-brittle end-to-end frontend integration and smoke tests.
* **Immediate E2E**: Write an E2E smoke test *immediately* after wiring up the frontend-to-backend proxy to catch fundamental Docker networking issues.
* **Visual Regression Testing (VRT)**: Use `.toHaveScreenshot()` with dynamic masks to assert visual styling stability.
