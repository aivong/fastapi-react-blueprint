---
name: fastapi-react-ci-cd
version: "1.0.0"
description: "Guidelines for maximizing CI/CD pipeline efficiency and developer velocity."
tags:
  - ci-cd
  - testing-economics
  - github-actions
  - automation
dependencies:
  - karpathy-guidelines
---

# CI/CD Pipeline & Testing Economics

Use this skill when designing, configuring, or modifying CI/CD workflows (e.g., GitHub Actions) and optimizing local developer testing feedback loops.

## 1. Dual-Path Testing Loop

Long-running integration tests (like Testcontainers) are invaluable for strict, clean-room validation, but they severely bottleneck developer velocity.
* You **MUST** establish a Dual-Path workflow using Pytest markers.
* Apply `@pytest.mark.integration` to all tests requiring a database or heavy containerized services.
* **Local Development**: Developers should default to running pure unit tests locally for instant feedback:
  ```bash
  pytest -m "not integration"
  ```
* **CI & Ad-hoc Run**: The slow, clean-room Testcontainers suite (`pytest` with no filter) is reserved for CI pipelines and explicit, pre-commit ad-hoc verifications.

---

## 2. CI Redundancy Elimination

CI pipelines must be aggressively optimized to conserve compute minutes and reduce developer wait times.
* Configure heavy E2E test suites to run **ONLY** on Pull Requests (`on: pull_request`).
* **Never** trigger the full, slow test suite on direct `push` to the `main` branch. Any commit landing on `main` has already proven its correctness by passing the PR gate.

---

## 3. Early PR Lint & Typecheck Gates

Optimize the feedback loop for contributors by splitting CI jobs based on execution time and cost.
* You **MUST** configure Pull Request CI gates to run lightweight static analysis as early as possible.
* Specifically, run:
  * **Frontend**: `npm run lint` and `npm run typecheck`
  * **Backend**: `ruff check` and typechecking
* Running these gates immediately on PR creation/update ensures that syntax, lint, or type safety issues are flagged before resource-heavy integration or E2E tests are scheduled to run.
