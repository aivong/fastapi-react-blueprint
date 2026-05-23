---
name: fastapi-react-blueprint
version: "1.0.0"
description: "The definitive architectural blueprint, design decisions, and skill stack for building modern, secure, and production-ready Python/React applications."
tags:
  - architecture
  - blueprint
  - stack
  - skills
---

# Modern Stack Architecture & Design Decisions

This skill encapsulates a curated set of design decisions for building modern, agent-driven full-stack applications. Load this skill when you need to understand the 'why' behind the technology stack and the rules governing development.

## 1. Chosen Technology Stack & Rationale

| Technology | Purpose | Why We Chose It |
|---|---|---|
| **Python (FastAPI)** | Backend & Orchestrator Daemon | Python is the standard for AI/agentic tooling (like `antigravity`). FastAPI provides immense speed, asynchronous support, and native OpenAPI/Swagger generation. |
| **Latest Node.js LTS (via nvm)** | Frontend Dashboard | Remix/React Router provides a robust SPA architecture. We mandate using **nvm** to manage Node versions locally and enforce using the **latest Node.js LTS version** (e.g., v24, v26, etc.) for all frontend compilation. |
| **PostgreSQL & SQLModel** | Database Persistence | PostgreSQL is the most robust open-source relational database. SQLModel (by the creator of FastAPI) combines SQLAlchemy 2.0 and Pydantic into a single class, eliminating the need to duplicate schemas for the API and the Database. |
| **Testcontainers** | Spec-Driven Acceptance Tests | Allows us to spin up isolated PostgreSQL databases and FastAPI servers in Docker containers during testing, ensuring zero state bleeding between tests. |
| **PyInstaller (Optional: Inno Setup)** | Cross-Platform Distribution | Bundles the entire Python daemon, dependencies, and React static assets into a single native binary (macOS/Linux/Windows) for non-technical users. Windows builds can optionally use Inno Setup to compile a professional `.exe` setup wizard. |
| **Docker Compose** | Local Orchestration | Provides an easy `docker compose up` command to launch the database, API, and frontend simultaneously for local development. |

## 2. Integrated Skills & Rationale

We have aggressively integrated specialized skills to enforce a titanium-clad codebase. You MUST adhere to these skills during development:

### Core Philosophy
*   **`karpathy-guidelines`**: **Simplicity First** and **Surgical Changes**. We write the absolute minimum code required to solve the problem. Zero speculative features. We only touch the code we must touch.

### Security & Secrets
*   **Supply Chain Security**: All dependencies in `pyproject.toml` and `package.json` MUST be pinned to exact versions (e.g., `fastapi==0.136.1` instead of `>=`). Lockfiles (`uv.lock`, `package-lock.json`) MUST be committed to version control to eliminate vector supply chain attacks.
*   **Minimal Attack Surfaces**: Always use `slim` or `alpine` variants for Docker base images (e.g., `python:3.12-slim`, `postgres:17-alpine`) to minimize footprint and reduce vulnerability exposure.
*   **`owasp-secure-coding`**: Security is built-in by default (preventing XSS, Injection, SSRF) before code ever reaches production.
*   **`varlock`**: Strict secret management. API keys (`LINEAR_API_KEY`) are injected via `.env` files and never logged or hardcoded.
*   **`shannon`**: Autonomous AI Pentesting. A developer can trigger this skill at any time to execute real white-box exploits against the local Docker Compose environment to prove the app is secure.

### Database & Migrations
*   **`postgres-best-practices` (PlanetScale)**: Guides our schema design, optimal indexing strategies, and connection troubleshooting.
*   **`alembic-safe-migrations`**: Enforces strict database migration rules: always implement `downgrade()`, add columns as `nullable` first to avoid locking, and separate data migrations from schema migrations.

### API & Application Architecture
*   **`twelve-factor`**: Enforces stateless execution and Graceful Shutdowns. The orchestrator must intercept `SIGINT`/`SIGTERM` to safely close active agent sub-processes.
*   **`api-design`**: Enforces strict REST semantics and standardized RFC 9457 error shapes across all FastAPI endpoints.
*   **OpenAPI Autogeneration (`openapi-typescript`)**: Enforces 100% type safety across the frontend/backend boundary. Instead of brittle, manual interface definitions, the React frontend must generate its schemas directly from FastAPI's `/openapi.json` to prevent schema drift and catch API contract breakages at compile-time.
*   **`frontend-design`**: Enforces an "Industrial / Utilitarian Control Panel" aesthetic using Tailwind v4 and shadcn/ui.

### Testing & Quality Assurance

1. **Test-Driven Development (TDD)**: ALL code must be written using strict Red-Green-Refactor cycles.
2. **Early E2E Smoke Tests**: Write a Playwright E2E smoke test *immediately* after wiring up the frontend-to-backend proxy. This catches fundamental Docker networking and Vite host-mapping issues before asking the user to manually verify the UI.
3. **OpenAPI Contract Safety**: Do not manually write frontend API clients or TypeScript types. Use `openapi-typescript` to autogenerate types from FastAPI's OpenAPI schema, and `openapi-fetch` to ensure compile-time contract safety.
4. **Chaos Testing (Graceful Degradation)**: Use the `respx` library to intentionally simulate catastrophic upstream failures (502 Bad Gateway, 504 Gateway Timeout) from external dependencies. **Critical**: When testing FastAPI apps with `respx`, avoid `TestClient` (which spawns isolated threads that break `respx` contextvars). Instead, run async tests using `httpx.AsyncClient(transport=httpx.ASGITransport(app=app))`.
5. **Visual Regression Testing (VRT)**: Use Playwright's `.toHaveScreenshot()` to capture pixel-perfect baselines of critical UI components to catch CSS regressions during refactors.
6. **Mutation Testing**: Use `mutmut` to verify the strength of the test suite. **Critical Lesson (The Pythonic Refactor)**: Avoid using the `src.` prefix in Python imports (e.g. do not use `from src.models import Issue`). Instead, use relative or direct imports (`from models import Issue`) and append `src` to the `PYTHONPATH`. This prevents `mutmut` from crashing due to namespace package compilation bugs. Note that running mutation tests that rely on `testcontainers` from inside a Docker container requires mounting `/var/run/docker.sock`.

### Documentation & Demonstration
*   **`excalidraw-diagram`**: Used to generate isomorphic visual arguments (system architecture diagrams) that are saved directly to version control.
*   **`remotion-best-practices`**: Used at the very end of the project to programmatically generate a highly polished demo MP4 video of the application in action without using CSS animations.
