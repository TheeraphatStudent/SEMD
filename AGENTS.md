# Repository Guidelines

## Project Structure & Module Organization

SEMD is an umbrella repository for a malicious URL checker. The intended layout is:

- `semd-backend/`: Python/FastAPI backend; expected folders include `config/`, `database/`, `models/`, `routers/`, `services/`, and `workers/`.
- `semd-ml/`: ML service; expected locations include `src/`, `models/`, `reports/`, `requirements.txt`, and container files.
- `semd-frontend/`: Next.js/React frontend; expected locations include `src/`, `public/`, `package.json`, Tailwind config, and Next config.
- `semd-extension/`: Browser extension; expected locations include `chrome_extension/`, `firefox_addons/`, `scripts/`, and `extension.conf.yaml`.
- `README.md`: setup notes, ports, and architecture overview.

## Build, Test, and Development Commands

Run commands from the relevant component directory.

- `uv python install 3.12`: install the Python version used during development.
- `uv sync`: create or update the local virtual environment.
- `uv run pyformat --in-place **/*.py`: format Python files before committing.
- `uv run pytest`: run Python tests for backend or ML components.
- `podman network create semd-shared-network`: create the shared local service network.
- `npm install`: install frontend or extension dependencies when a `package.json` is present.
- `npm run dev`, `npm test`, `npm run build`: run component-defined development, test, and build scripts.

Service ports documented in `README.md`: backend `3000`, frontend `3001`, MLflow `5000`, Redis `6379`, PostgreSQL `5432`.

## Coding Style & Naming Conventions

Use Python `3.12.x` with `uv` for backend and ML work. Keep Python modules and files in `snake_case`; use explicit service, router, and worker names. For React code, prefer component files in `PascalCase` and hooks/utilities in `camelCase` or `kebab-case`. Keep configuration in component-specific files rather than hard-coding ports or secrets.

## Testing Guidelines

Place tests near the component they validate. For Python services, use `test_*.py` naming and run them with `uv run pytest`. For frontend or extension tests, follow the package's configured test runner. Run relevant tests before opening a PR.

## Commit & Pull Request Guidelines

Recent history uses short conventional-style messages such as `feat: init parent` and `cleanup: remove submodule references`. Keep commits concise and imperative, for example `fix: validate URL payload`.

Pull requests should include a clear summary, test results, linked issues when applicable, and screenshots for UI changes. Note service, port, environment, or migration changes explicitly.

## Security & Configuration Tips

Do not commit secrets, credentials, sensitive model artifacts, or local environment files. Document required variables in the relevant component README or example env file.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

When the user types `/graphify`, invoke the `skill` tool with `skill: "graphify"` before doing anything else.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- Dirty graphify-out/ files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify. Only skip graphify if the task is about stale or incorrect graph output, or the user explicitly says not to use it.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
