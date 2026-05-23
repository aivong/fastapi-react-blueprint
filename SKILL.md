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
*   **`hexagonal-architecture`**: Enforces a strict Ports and Adapters architecture to completely decouple core Domain logic from Infrastructure concerns (like Databases, third-party APIs, and external services). This guarantees that all external dependencies remain hot-swappable via Adapters, and enables robust, behavior-driven integration testing using Fakes instead of brittle mocks.
*   **`twelve-factor`**: Enforces stateless execution and Graceful Shutdowns. The orchestrator must intercept `SIGINT`/`SIGTERM` to safely close active agent sub-processes.
*   **`api-design`**: Enforces strict REST semantics and standardized RFC 9457 error shapes across all FastAPI endpoints.
*   **OpenAPI Autogeneration (`openapi-typescript`)**: Enforces 100% type safety across the frontend/backend boundary. Instead of brittle, manual interface definitions, the React frontend must generate its schemas directly from FastAPI's `/openapi.json` to prevent schema drift and catch API contract breakages at compile-time.
*   **`frontend-design`**: Enforces an "Industrial / Utilitarian Control Panel" aesthetic using Tailwind v4 and shadcn/ui.

### Testing & Quality Assurance: The 5-Tier "Shift Left" Pyramid

We aggressively shift left to eliminate manual testing, employing a 5-tier pyramid:

1. **Tier 0: Static Analysis (Pre-Test)**: Use Astral's `ty` to catch type mismatches and `ruff` to catch logical flaws before tests even execute.
2. **Tier 1: Unit & Mutation Tests**: Write fast, isolated tests for Domain logic using strict TDD. Crucially, run **Mutation Testing (`mutmut`)** against this tier to prove the test suite actually catches injected bugs. (Note: use relative/direct imports `from models import Issue` rather than `src.models` to prevent `mutmut` namespace bugs).
3. **Tier 2: Contract Tests (Schema Validation)**: Never guess if an external API changed. For GraphQL APIs, extract queries into shared `.graphql` files (treated as immutable source code, never Application State) and use the frontend's `@graphql-codegen/cli` to validate them against the live production schema during CI.
4. **Tier 3: Component Integration (Testcontainers & Fakes)**: Test infrastructure adapters using real backing services. Use `testcontainers-python` to spin up ephemeral PostgreSQL databases. For external APIs, **build Fakes instead of Mocks** using `respx`. Fakes maintain in-memory state dictionaries and test the *behavior* of the orchestrator, unlike brittle mocks that only assert `called_with()`.
5. **Tier 4: Automated E2E System Tests**: Programmatically execute the entire orchestrator loop and mathematically assert the outcomes (e.g., executing the agent's generated code via `subprocess` to prove it runs) to completely eliminate manual UI verification.

*Note on Playwright*: Write an E2E smoke test *immediately* after wiring up the frontend-to-backend proxy to catch fundamental Docker networking issues. Use `.toHaveScreenshot()` with dynamic masks for Visual Regression Testing (VRT).

### Documentation & Demonstration
*   **`excalidraw-diagram`**: Used to generate isomorphic visual arguments (system architecture diagrams) that are saved directly to version control.
*   **`remotion-best-practices`**: Used at the very end of the project to programmatically generate a highly polished demo MP4 video of the application in action without using CSS animations.
