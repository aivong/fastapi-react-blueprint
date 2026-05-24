---
name: fastapi-react-blueprint
version: "1.0.0"
description: "The definitive architectural blueprint, design decisions, and skill stack for building modern, secure, and production-ready Python/React applications."
tags:
  - architecture
  - blueprint
  - stack
  - skills
dependencies:
  - karpathy-guidelines
  - owasp-secure-coding
  - varlock
  - shannon
  - postgres-best-practices
  - alembic-safe-migrations
  - hexagonal-architecture
  - twelve-factor
  - api-design
  - frontend-design
  - excalidraw-diagram
  - remotion-best-practices
---

# Modern Stack Architecture & Design Decisions

This skill encapsulates a curated set of design decisions for building modern, agent-driven full-stack applications. Load this skill when you need to understand the 'why' behind the technology stack and the rules governing development.

## 1. Chosen Technology Stack & Rationale

| Technology | Purpose | Why We Chose It |
|---|---|---|
| **Python (FastAPI)** | Backend & Orchestrator Daemon | Python is the standard for AI/agentic tooling. FastAPI provides immense speed, asynchronous support, and native OpenAPI/Swagger generation. |
| **Astral Tooling (Ruff, UV, Ty)** | Static Analysis & Execution | We exclusively use Astral tooling for the Python ecosystem: `ruff` for lightning-fast linting/formatting, `uv` for package management, and `ty` for extremely fast, Rust-based static type checking. |
| **Frontend Architecture** | **React Router v7 (SPA Mode)** | Provides a robust, lightweight frontend architecture. For **all applications** where SEO and Server-Side Rendering (SSR) are unnecessary, always initialize using React Router v7 in SPA Mode (`npx create-react-router@latest` -> SPA). This provides Remix's powerful `loader`/`action` paradigms without the heavy Node server overhead. **Tailwind v4 & shadcn/ui**: All UI components must use the `frontend-design` skill aesthetic. Tailwind v4 must be the CSS engine. |
| **PostgreSQL & SQLModel** | Database Persistence | PostgreSQL is the most robust open-source relational database. **Always use the latest stable major version** (e.g., `postgres:18-alpine`). SQLModel (by the creator of FastAPI) combines SQLAlchemy 2.0 and Pydantic into a single class, eliminating the need to duplicate schemas for the API and the Database. |
| **Testcontainers** | Spec-Driven Acceptance Tests | Allows us to spin up isolated PostgreSQL databases and FastAPI servers in Docker containers during testing, ensuring zero state bleeding between tests. |
| **PyInstaller (Optional: Inno Setup)** | Cross-Platform Distribution | Bundles the entire Python daemon, dependencies, and React static assets into a single native binary (macOS/Linux/Windows) for non-technical users. Windows builds can optionally use Inno Setup to compile a professional `.exe` setup wizard. |
| **Docker Compose & Make** | Local Developer Experience (DX) | We enforce `docker-compose.yml` profiles (`profiles: ["full"]`) so that `docker compose up -d` only provisions the database. We use a `Makefile` to abstract the DX: `make dev` uses `npx concurrently` to boot the database and stream both the native Python and React hot-reloading servers in a single, unified terminal. |

## 2. Integrated Skills & Rationale

We have aggressively integrated specialized skills to enforce a titanium-clad codebase. You MUST adhere to these skills during development:

### Backend Structure (FastAPI + Hexagonal + CQRS)
*   **Composition Root Testing**: The Dependency Injection graph (the Composition Root typically found in `main.py`) MUST be encapsulated into a pure factory function (e.g., `build_daemon()`). You MUST write an integration test that simply calls this factory to mathematically assert that all positional/keyword arguments match the adapter signatures. Never leave the composition root untested.
*   **No Global State Initialization**: Never statically initialize stateful connection dependencies (like database `engine`s or `httpx` clients) at the global module level (e.g. `engine = create_engine(...)` at the top of `api.py`). Because Python aggressively caches module imports in `sys.modules`, a test suite that dynamically alters `os.environ` will cause silent, irreversible state drift across all subsequent tests. Always encapsulate stateful clients within factory functions or FastAPI's `Depends()` injection system.
*   **SQLModel over SQLAlchemy**: Use SQLModel to natively blend Pydantic schemas with SQLAlchemy models.

### Core Philosophy
*   **`karpathy-guidelines`**: **Simplicity First** and **Surgical Changes**. We write the absolute minimum code required to solve the problem. Zero speculative features. We only touch the code we must touch.

### Security & Secrets
*   **Supply Chain Security & Freshness**: All dependencies in `pyproject.toml` and `package.json` MUST be pinned to exact versions (e.g., `fastapi==0.136.1` instead of `>=`). Lockfiles (`uv.lock`, `package-lock.json`) MUST be committed to version control to eliminate vector supply chain attacks. Additionally, you must default to using the **latest stable major version** of all backing services (e.g., Node.js LTS, PostgreSQL 18) unless explicitly constrained by the user.
*   **Minimal Attack Surfaces**: Always use `slim` or `alpine` variants for Docker base images (e.g., `python:3.12-slim`, `postgres:17-alpine`) to minimize footprint and reduce vulnerability exposure.
*   **`owasp-secure-coding`**: Security is built-in by default (preventing XSS, Injection, SSRF) before code ever reaches production.
*   **`varlock`**: Strict secret management. API keys (`LINEAR_API_KEY`) are injected via `.env` files and never logged or hardcoded.
*   **`shannon`**: Autonomous AI Pentesting. A developer can trigger this skill at any time to execute real white-box exploits against the local Docker Compose environment to prove the app is secure.

### Database & Migrations
*   **`postgres-best-practices` (PlanetScale)**: Guides our schema design, optimal indexing strategies, and connection troubleshooting.
*   **`alembic-safe-migrations`**: Enforces strict database migration rules: always implement `downgrade()`, add columns as `nullable` first to avoid locking, and separate data migrations from schema migrations.

### API & Application Architecture
*   **`hexagonal-architecture`**: Enforces a strict Ports and Adapters architecture to completely decouple core Domain logic from Infrastructure concerns (like Databases, third-party APIs, and external services). This guarantees that all external dependencies remain hot-swappable via Adapters, and enables robust, behavior-driven integration testing using Fakes instead of brittle mocks.
*   **External API Resilience (Rate Limiting & Permissions)**: Any Infrastructure Adapter integrating with a 3rd-party external service (e.g. Linear, GitHub) MUST be resilient to misconfigurations and throttling. 
    *   **Rate Limits:** If an API returns an HTTP 429 (Too Many Requests), the adapter must parse the `Retry-After` or rate-limit reset headers and gracefully sleep or back-off until the limit resets. Do not blindly hammer external APIs. 
    *   **Permissions:** Prefer handling read-only or lower-permission API keys gracefully (e.g., catching HTTP 401/403 Forbidden). Instead of crashing the application on a write mutation, intercept the error, log a warning, and safely skip the action. This ensures the app doesn't fatally crash if a user misconfigures their external service permissions.
*   **Resource Cleanup & Disk Leaks**: For code involving real file creation or physical workspaces, the application MUST implement strict cleanup mechanisms (e.g., using `try...finally` blocks) to prevent disk space leaks over time. During tests, either mathematically assert these cleanup mechanisms or utilize Pytest's built-in `tmp_path` fixture to guarantee automatic file cleanup.
*   **Background Worker Observability**: When dynamically generating or spawning isolated background subprocesses or worker scripts, you MUST design for rigorous observability upfront to prevent silent failures. 
    *   **Isolate Logs**: Never hardcode diagnostic log filenames (e.g., `error.log`). Always append a UUID or `task_id` to prevent concurrent test executions from silently overwriting each other's diagnostic data.
    *   **Force Buffer Flushes**: When writing to diagnostic files from inside a fragile script, explicitly call `f.flush()` after writing. If the worker script hard-crashes before the file handler safely closes, the Python IO buffer will be lost, resulting in an impossible-to-debug 0-byte log file.
    *   **Never Swallow Stderr**: Never pipe `subprocess.DEVNULL` to `stderr` unless absolutely necessary. Explicitly redirect the raw `stderr` stream to an isolated diagnostic log file on disk to guarantee the capture of native OS crashes and Python stack traces.
*   **Stale Artifact Cache Invalidation Testing**: When an application dynamically generates scripts, code, or configuration files to disk, you MUST write isolated unit tests that explicitly guard against stale caching regressions. These tests must proactively seed the destination path with a "stale garbage" file, trigger the generation logic, and mathematically assert that the garbage was completely overwritten. Clean-slate CI/CD test environments naturally mask these local-caching bugs, making intentional cache-poisoning tests mandatory.
*   **`twelve-factor`**: Enforces stateless execution and Graceful Shutdowns. The orchestrator must intercept `SIGINT`/`SIGTERM` to safely close active agent sub-processes.
*   **`api-design`**: Enforces strict REST semantics and standardized RFC 9457 error shapes across all FastAPI endpoints.
*   **OpenAPI Autogeneration (`openapi-typescript`)**: Enforces 100% type safety across the frontend/backend boundary. Instead of brittle, manual interface definitions, the React frontend must generate its schemas directly from FastAPI's `/openapi.json` to prevent schema drift and catch API contract breakages at compile-time.
*   **`frontend-design`**: Enforces an "Industrial / Utilitarian Control Panel" aesthetic using Tailwind v4 and shadcn/ui.

### Testing & Quality Assurance: The 5-Tier "Shift Left" Pyramid

We aggressively shift left to eliminate manual testing, employing a 5-tier pyramid:

1. **Tier 0: Static Analysis (Pre-Test)**: Use Astral's `ty` to catch type mismatches and `ruff` to catch logical flaws before tests even execute.
2. **Tier 1: Unit & Mutation Tests**: Write fast, isolated tests for Domain logic using strict TDD. Crucially, run **Mutation Testing (`mutmut`)** against this tier to prove the test suite actually catches injected bugs. (Note: use relative/direct imports `from models import Issue` rather than `src.models` to prevent `mutmut` namespace bugs).
3. **Tier 2: Contract Tests (Schema Validation)**: Never guess if an external API changed. For GraphQL APIs, extract queries into shared `.graphql` files (treated as immutable source code, never Application State) and use the frontend's `@graphql-codegen/cli` to validate them against the live production schema during CI.
4. **Tier 3: Component Integration (Testcontainers & Fakes)**: Test infrastructure adapters using real backing services. Use `testcontainers-python` to spin up ephemeral PostgreSQL databases. 
    *   *Virtualization Tolerance*: When writing asynchronous polling loops backed by Testcontainers (especially on Windows/macOS Docker Desktop VMs), always configure generous timeouts (e.g., 20.0s). The initial TCP handshake through the VM NAT bridge often experiences 5-10 second connection delays that will flake brittle timeouts.
    *   *Fakes over Mocks*: For external APIs, **build Fakes instead of Mocks** using `respx`. Fakes maintain in-memory state dictionaries and test the *behavior* of the orchestrator, unlike brittle mocks that only assert `called_with()`.
5. **Tier 4: Automated E2E System Tests**: Programmatically execute the entire orchestrator loop and mathematically assert the outcomes (e.g., executing the agent's generated code via `subprocess` to prove it runs) to completely eliminate manual UI verification.

*Note on Playwright*: Write an E2E smoke test *immediately* after wiring up the frontend-to-backend proxy to catch fundamental Docker networking issues. Use `.toHaveScreenshot()` with dynamic masks for Visual Regression Testing (VRT).

### Continuous Integration & Testing Economics
*   **Dual-Path Testing Loop**: Long-running integration tests (like Testcontainers) are invaluable for strict, clean-room validation, but they severely bottleneck developer velocity. You MUST establish a Dual-Path workflow using Pytest markers. Apply `@pytest.mark.integration` to all tests requiring a database. Developers should default to running pure unit tests locally (`pytest -m "not integration"`) for instant feedback. The slow, clean-room Testcontainers suite (`pytest`) is reserved for CI pipelines and explicit ad-hoc verifications. **Never** use `USE_LOCAL_DB` environment variables to dynamically repoint test database URLs, as this pollutes local state and masks virtualization networking issues.
*   **CI Redundancy Elimination**: CI pipelines must be aggressively optimized to conserve compute minutes. The heavy E2E test suite should be configured to run **ONLY** on Pull Requests (`on: pull_request`). Never trigger the test suite on `push` to the `main` branch, as any commit landing on `main` has already mathematically proven its correctness during the PR gate.

### Documentation & Demonstration
*   **`excalidraw-diagram`**: Used to generate isomorphic visual arguments (system architecture diagrams) that are saved directly to version control.
*   **`remotion-best-practices`**: Used at the very end of the project to programmatically generate a highly polished demo MP4 video of the application in action without using CSS animations.
