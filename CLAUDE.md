# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository shape

SEMD ("Suspicious-URL Evaluation for Malicious Detection") is a malicious-URL checker split across four independent git submodules, each with its own remote and history:

| Module | Path | Stack | Remote |
|---|---|---|---|
| Backend API | `semd-backend/` | Python / FastAPI / SQLAlchemy / PostgreSQL / Redis | `TheeraphatStudent/SEMD-backend` |
| ML service | `semd-ml/` | Python / classical ML (sklearn, XGBoost) / MLflow / Redis | `TheeraphatStudent/SEMD-ml` |
| Web frontend | `semd-frontend/` | Next.js 14 / React 18 / Tailwind | `TheeraphatStudent/SEMD-frontend` |
| Browser extension | `semd-extension/` | Next.js 14 popup app + Chrome/Firefox manifest builds | `TheeraphatStudent/SEMD-extension` |

Because these are separate repos, changes inside a module must be committed inside that module's own working tree — a commit made from the root repo will not capture module-internal work, and vice versa. Don't assume a single `git log`/`git status` at root reflects module history; check inside the relevant `semd-*/` directory.

Python modules (`semd-backend/`, `semd-ml/`) are meant to be run with `uv`, but neither currently has a `pyproject.toml`/`uv.lock` — only `requirements.txt`. `uv sync` needs a `pyproject.toml` to work, so run `uv init --no-readme` (or `uv add -r requirements.txt` to generate one from the existing lockfile-less deps) before `uv sync` will succeed for the first time in either module.

`semd-ml/` has its own detailed `CLAUDE.md` (module structure, data flow, config-file coupling) — read it before working in that module instead of re-deriving it here.

Clone with `git clone --recurse-submodules`, or after a plain clone run `git submodule update --init --recursive`.

## Cross-service architecture

The prediction path crosses all of backend and ML:

```
semd-frontend / semd-extension
    → semd-backend (FastAPI, routers/prediction/prediction_route.py)
    → services/ml_prediction_service.py → services/ml_service_client.py
    → Redis queue "ml_prediction_queue"
    → semd-ml worker container (workers/queue_worker.py → ml/prediction_service.py, uses ml/ml_pipeline.py)
    → Redis cache "ml_result:{job_id}"
    → ml_service_client.py polls the cache
    → response back to caller
```

All four services share one podman/docker bridge network, `semd-shared-network`, created once before running any compose file:

```bash
podman network create semd-shared-network
```

Service ports: backend `3000`/`8000` (docs at `/docs`, `/redoc`; OpenAPI at `semd-backend/openapi.yaml`), frontend `3001`, MLflow `5000`, Redis `6379`, PostgreSQL `5432`.

Third-party URL-reputation services (Cloudflare Radar URL Scanner, Thai PhishTank) are registered dynamically at runtime via `POST /setting/third-service` rather than hardcoded — see `semd-backend/README.md` for the exact payloads.

## semd-backend

Layered structure: `routers/` (FastAPI route registration, one subpackage per domain: `auth`, `ml`, `prediction`, `report`, `setting`, `stat`, `dashboard`, `queue`) → `control/` (business logic, one `*Control` class per domain) → `services/` (external calls: DB, Redis, third-party HTTP) → `models/` (Pydantic request/response schemas) → `database/` (SQLAlchemy models, `database/models.py`). All routers inherit from `routers/base_route.py`'s `BaseRoute`; all response schemas inherit from `models/base_response_model.py`'s `BaseResponseModel` — high fan-in on these two is expected, not a coupling problem.

Auth: `guard/auth_guard.py`'s `AuthGuard` verifies `x-api-key` header and/or Bearer token as FastAPI dependencies; `get_db()`/`get_async_db()` in the same file yield SQLAlchemy sessions (sync and async) via `services/client/postgres_client.py`.

Setup and run (from `semd-backend/`), using `uv` — Python `3.12.x`:
```bash
uv python install 3.12
uv sync
cp .env.example .env
uv run fastapi dev main.py           # dev server, or:
uv run uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```
`backend-working.sh` (plain venv + pip, no `uv`) also exists and is what the module's own README documents — prefer the `uv` flow above. On WSL/Linux, if `backend-working.sh` fails with a CRLF error, run `sed -i 's/\r$//' ./backend-working.sh` first.

Docker: `docker compose -f compose.yaml up -d` (includes `database/compose.yaml` for Postgres/Redis; depends on `semd-shared-network` existing).

No test suite currently exists in this module (no `test_*.py`, no pytest config).

## semd-ml

See `semd-ml/CLAUDE.md` for the full module map and data flow. Quick reference, using `uv` — Python `3.12.x`:
```bash
uv python install 3.12
uv sync
cp .env.example .env
cd src
uv run main.py data-migrate            # populate dataset/ from archives, run once
uv run main.py train --dataset-files dataset/raw --algorithms decision_tree random_forest xgboost svm --run-name my_run
uv run main.py predict --url "https://example.com" --model-id <run_id>
uv run main.py worker --mode combined  # consumes ml_training_queue / ml_prediction_queue
uv run verify_imports.py               # smoke-tests that every module imports cleanly (not an architecture signal — its cross-module import edges are expected)
```
`docker-compose up -d` starts the ML service plus an MLflow server (image `ghcr.io/mlflow/mlflow:v3.10.0`, UI at `http://localhost:5000`). If MLflow permission errors appear, `sudo chown -R 1001:1001 /home/semd/.mlflow` (or run `setup_mlflow_permissions.sh`).

No test suite currently exists in this module.

## semd-frontend

Next.js 14 App Router with route groups: `src/app/(admin)`, `src/app/(auth)`, `src/app/(dashboard)`, plus `src/app/predict` and `src/app/users`. Shared UI primitives (`Card`, `CardHeader`, `CardContent`, etc.) live in `src/components/ui/`; their high fan-in across pages is expected.

The typed API client (`src/services/generated/`) is code-generated from the backend's OpenAPI spec via `orval` (config in `orval.config.ts`) — run `npm run generate:api` after the backend's `openapi.yaml` changes rather than hand-editing generated clients. Note both `src/lib/` and `src/libs/` exist side by side; check which one a given helper actually lives in before assuming a path.

```bash
npm install
npm run dev      # localhost:3000
npm run build
npm run lint
npm run generate:api
```
No `test` script is currently defined in `package.json`.

## semd-extension

A single Next.js app (`app/`) is compiled into two separate browser packages via `scripts/build-extension.js`: `chrome_extension/` and `firefox_addons/` are **build output**, not hand-edited source — edit `app/` and the manifest templates, then rebuild.

```bash
npm install
npm run build:chrome    # -> chrome_extension/
npm run build:firefox   # -> firefox_addons/
npm run build:all        # both
npm run clean            # removes out/dist/.next/builds/node_modules and both built extension dirs
```
Load `chrome_extension/` unpacked at `chrome://extensions/`; load `firefox_addons/manifest.json` at `about:debugging#/runtime/this-firefox`. No `test` script is currently defined.

## graphify (knowledge graph)

This project maintains a knowledge graph at `graphify-out/` (god nodes, community structure, cross-file/cross-module relationships spanning all four submodules).

When the user types `/graphify`, invoke the `skill` tool with `skill: "graphify"` before doing anything else.

- For codebase questions, first run `graphify query "<question>"` when `graphify-out/graph.json` exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts — these return a scoped subgraph, usually much smaller than `GRAPH_REPORT.md` or raw grep output.
- Dirty `graphify-out/` files are expected after hooks or incremental updates; that alone is not a reason to skip graphify. Only skip it if the task is about stale/incorrect graph output, or the user explicitly says not to use it.
- If `graphify-out/wiki/index.md` exists, use it for broad navigation instead of raw source browsing.
- Read `graphify-out/GRAPH_REPORT.md` only for broad architecture review, or when query/path/explain don't surface enough context. Treat its "god node" and "surprising connection" findings as hypotheses to verify against actual source, not conclusions — several past findings turned out to be either legitimate shared base classes/UI primitives (expected high fan-in) or artifacts of `verify_imports.py`'s smoke-test imports (not real coupling).
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
