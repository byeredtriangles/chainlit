# AGENTS

# =============================================================================
# Purpose
# -----------------------------------------------------------------------------
# This file tells AI code agents (e.g. OpenAI Codex) **exactly** how to work
# with the Chainlit code-base.  It covers environment setup, build/test
# commands, code style, commit conventions, and project structure so agents
# can reason and act without guesswork.
#
# If you are a human contributor, CONTRIBUTING.md contains similar information
# with extra context and explanations.
# =============================================================================

## 0. Quick Start Checklist
1. **Clone** the user’s fork (replace `$FORK`):
   ```bash
   git clone https://github.com/$FORK/chainlit.git && cd chainlit
  ```

2. **Set up** Python + Node workspace:

   ```bash
   cd backend && poetry install --with tests --with mypy --with dev
   cd ../frontend && pnpm install
   cd ..            # back to repo root
   ```
3. **Run everything**:

   ```bash
   # Terminal 1 – backend server (headless)
   cd backend && poetry run chainlit run chainlit/hello.py -h
   # Terminal 2 – frontend dev server
   cd frontend && pnpm dev --port 5174 --host
   # Visit: http://localhost:5174
   ```
4. **Test before commit**:

   ```bash
   # Backend unit tests
   cd backend && poetry run pytest
   # E2E (Cypress) tests – may take a while
   pnpm test
   ```
5. **Format & lint**:

   ```bash
   pnpm formatPython  # Black + isort
   pnpm lint          # ESLint + Prettier for TS/JS
   ```

---

## 1. Project Overview

| Path                | What lives here                           | Main language / tooling |
| ------------------- | ----------------------------------------- | ----------------------- |
| `backend/`          | Chainlit Python package & CLI             | Python 3.10+, Poetry    |
| `frontend/`         | React/Vite UI (served by backend in prod) | TS/JS, pnpm, Vite       |
| `cypress/`          | E2E tests (Cypress)                       | JavaScript              |
| `examples/`         | Minimal demo apps                         | Python                  |
| `.github/workflows` | CI for tests & linting                    | GitHub Actions          |

---

## 2. Environment Specifications

### Python

* **Version** ≥ 3.10
* **Manager** Poetry (pyproject.toml is canonical).
* **Virtual env activation** is automated by `poetry shell`.

### Node / JS

* **Version** ≥ 16.x (LTS preferred).
* **Package manager** `pnpm` (monorepo workspaces).

> **Windows quirk**
> If `pnpm` post-install scripts fail, set script-shell to Git Bash:
> `pnpm config set script-shell "C:\\Program Files\\git\\bin\\bash.exe"`

---

## 3. Key Commands for Codex

| Task                                     | Command (run from repo root unless noted)                                                            |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Install all deps                         | `cd backend && poetry install --with tests --with mypy --with dev && cd ../frontend && pnpm install` |
| Run backend server                       | `cd backend && poetry run chainlit run chainlit/hello.py [-h]`                                       |
| Run backend server against custom target | `poetry run chainlit run path/to/app.py [-h]`                                                        |
| Run UI dev server                        | `cd frontend && pnpm dev --port 5174 --host`                                                         |
| Build production UI                      | `cd frontend && pnpm build`                                                                          |
| Backend unit tests                       | `cd backend && poetry run pytest`                                                                    |
| Full E2E tests                           | `pnpm test`                                                                                          |
| Single Cypress test                      | `SINGLE_TEST=<folder> pnpm test`                                                                     |
| Headed Cypress debug                     | `SINGLE_TEST=<folder> CYPRESS_OPTIONS="--headed --no-exit" pnpm test`                                |
| Auto-format Python                       | `pnpm formatPython`                                                                                  |
| Lint / format JS & TS                    | `pnpm lint && pnpm format`                                                                           |

---

## 4. Coding & Style Conventions

### Python

* **Formatter** Black, line length = 88.
* **Imports** isort (`pnpm formatPython` runs both).
* **Typing** `mypy` strict mode; prefer explicit `async` / `await`.

### JavaScript / TypeScript

* **Formatter** Prettier (via ESLint plugin).
* **Lint rules** ESLint + Airbnb/React base config.
* **React** Functional components; hooks over classes.

### Git / Commits

* Conventional Commits style (`feat: …`, `fix: …`, `test: …`).
* Each commit **must pass** `pytest` and `pnpm test` locally.

---

## 5. Testing Policy

1. **Backend PRs** must add or update relevant `tests/**` or `backend/tests/**`.
2. **UI PRs** that touch user-visible behaviour should add/adjust a Cypress spec.
3. **Long-running E2E** tests are allowed; CI handles caching.

---

## 6. Runtime Notes for Agents

* The backend **serves** the built frontend in production; during dev we run two
  processes (see Quick Start §0).
* Default backend **port** = `8000`; default UI dev port = `5174`.
* The backend exposes a FastAPI app; if you add routes, mount under `/api/*`
  to avoid conflicting with WebSocket paths.
* Chainlit’s CLI auto-reloads on file changes.

---

## 7. Common Pitfalls Codex Should Avoid

| Pitfall                                                           | Preventive Guideline                                                                 |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Forgetting to rebuild UI after backend-only edit                  | Not required – Vite dev server handles HMR.                                          |
| Breaking circular async loop between Chainlit websocket callbacks | Never call `cl.run_sync` inside an `@cl.on_message` coroutine.                       |
| Committing secrets                                                | Never commit `.env`, `openai_api_key`, or Clerk secrets.                             |
| Platform-specific paths in scripts                                | Use *POSIX-style* (`/`) paths inside pnpm scripts; Windows users set `script-shell`. |

---

## 8. Automation Hooks (CI)

The CI pipeline (`.github/workflows/ci.yml`) runs:

1. `pnpm lint`
2. `backend/scripts/typecheck.sh` (mypy)
3. `pytest` (unit)
4. `pnpm test` (Cypress e2e, headless)
5. `poetry run python -m pip check` (dependency health)

**Codex MUST ensure a PR passes all steps.**

---

## 9. Asking For Human Input

When the agent is uncertain about requirements or business logic:

1. **Pause.**
2. Comment in the PR describing the ambiguity.
3. Assign `@davidfurman` (or repository owner) for clarification.
   Never guess silently.

---

## 10. Useful Shortcuts

```bash
# Quickly start demo app with auto-reload
pnpm demo         # alias for `pnpm --filter demo-* dev`

# Recreate full dev environment from scratch
./scripts/dev_bootstrap.sh
```

---

## 11. Glossary

| Term                | Meaning                                                                    |
| ------------------- | -------------------------------------------------------------------------- |
| **Chainlit**        | The Python package (backend) plus React UI enabling LLM chat apps.         |
| **LiteralAI Cloud** | Optional SaaS offering for hosted analytics; **not** required for OSS dev. |
| **Headless** (`-h`) | CLI flag to launch backend without serving the built UI (dev convenience). |
| **HMR**             | Hot-module reload (frontend auto-refresh during development).              |

---

*Generated for Codex on 20 June 2025 – maintain alongside `CONTRIBUTING.md`.*
