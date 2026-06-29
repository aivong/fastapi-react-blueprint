---
name: fastapi-react-backend
version: "1.0.0"
description: "Architecture, logic, resilience, and testability patterns for robust FastAPI backends."
tags:
  - fastapi
  - backend
  - python
  - hexagonal
  - cqrs
dependencies:
  - karpathy-guidelines
  - owasp-secure-coding
  - varlock
  - postgres-best-practices
  - alembic-safe-migrations
  - hexagonal-architecture
  - twelve-factor
  - api-design
---

# FastAPI Backend Architecture & Implementation Guidelines

Use this skill when developing, refactoring, or reviewing the backend codebase of a Python/FastAPI application. These rules enforce reliability, clean boundaries, testability, and resilience.

## 1. Structure & Dependency Injection

### Composition Root Testing
* The Dependency Injection graph (the Composition Root typically found in `main.py`) **MUST** be encapsulated into a pure factory function (e.g., `build_daemon()`).
* You **MUST** write an integration test that simply calls this factory to mathematically assert that all positional/keyword arguments match the adapter signatures. Never leave the composition root untested.

### No Global State Initialization
* Never statically initialize stateful connection dependencies (like database `engine`s or `httpx` clients) at the global module level (e.g., `engine = create_engine(...)` at the top of `api.py`).
* Because Python aggressively caches module imports in `sys.modules`, a test suite that dynamically alters `os.environ` will cause silent, irreversible state drift across all subsequent tests.
* Always encapsulate stateful clients within factory functions or FastAPI's `Depends()` injection system.

### SQLModel over SQLAlchemy
* Use SQLModel to natively blend Pydantic schemas with SQLAlchemy models. This eliminates duplication between API models and database structures.

---

## 2. External API Resilience (Rate Limiting & Permissions)

Any Infrastructure Adapter integrating with a third-party external service (e.g., Linear, GitHub) **MUST** be resilient to misconfigurations and throttling.

* **Rate Limits**: If an API returns an HTTP 429 (Too Many Requests), the adapter must parse the `Retry-After` or rate-limit reset headers and gracefully sleep or back-off until the limit resets. Do not blindly hammer external APIs.
* **Permissions**: Prefer handling read-only or lower-permission API keys gracefully (e.g., catching HTTP 401/403 Forbidden). Instead of crashing the application on a write mutation, intercept the error, log a warning, and safely skip the action. This ensures the app doesn't fatally crash if a user misconfigures their external service permissions.

---

## 3. Background Worker Observability

When dynamically generating or spawning isolated background subprocesses or worker scripts, you **MUST** design for rigorous observability upfront to prevent silent failures.

* **Isolate Logs**: Never hardcode diagnostic log filenames (e.g., `error.log`). Always append a UUID or `task_id` to prevent concurrent test executions from silently overwriting each other's diagnostic data.
* **Force Buffer Flushes**: When writing to diagnostic files from inside a fragile script, explicitly call `f.flush()` after writing. If the worker script hard-crashes before the file handler safely closes, the Python IO buffer will be lost, resulting in an impossible-to-debug 0-byte log file.
* **Never Swallow Stderr**: Never pipe `subprocess.DEVNULL` to `stderr` unless absolutely necessary. Explicitly redirect the raw `stderr` stream to an isolated diagnostic log file on disk to guarantee the capture of native OS crashes and Python stack traces.

---

## 4. Resource Cleanup & Cache Invalidation

### Resource Cleanup & Disk Leaks
* For code involving real file creation or physical workspaces, the application **MUST** implement strict cleanup mechanisms (e.g., using `try...finally` blocks) to prevent disk space leaks over time.
* During tests, either mathematically assert these cleanup mechanisms or utilize Pytest's built-in `tmp_path` fixture to guarantee automatic file cleanup.

### Stale Artifact Cache Invalidation Testing
* When an application dynamically generates scripts, code, or configuration files to disk, you **MUST** write isolated unit tests that explicitly guard against stale caching regressions.
* These tests must proactively seed the destination path with a "stale garbage" file, trigger the generation logic, and mathematically assert that the garbage was completely overwritten.
* Clean-slate CI/CD test environments naturally mask these local-caching bugs, making intentional cache-poisoning tests mandatory.
