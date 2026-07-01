# SEMD

SEMD is a malicious URL checker integrated with machine learning. This root repository is intended to be used as the entry point for all project modules.

## Monorepo Access After Clone

Clone the root repository with submodules:

```bash
git clone --recurse-submodules https://github.com/TheeraphatStudent/SEMD.git
cd SEMD
```

If you already cloned the root repository and the module folders are empty, run:

```bash
git submodule update --init --recursive
```

After entering the root folder, access each project by changing into its module directory:

```bash
cd semd-backend     # Backend API
cd semd-ml          # Machine learning service
cd semd-frontend    # Web frontend
cd semd-extension   # Browser extension
```

Use `cd ..` to return to the root before entering another module. Do not clone another `SEMD` inside a module folder; the module source is provided by the configured submodules.

## Project Structure

```text
SEMD/
|-- semd-backend/      # Python/FastAPI backend service
|-- semd-ml/           # Python machine learning service
|-- semd-frontend/     # Next.js/React frontend
|-- semd-extension/    # Chrome/Firefox browser extension
|-- AGENTS.md          # Contributor and agent guidelines
`-- README.md          # Root project documentation
```

## Service Ports

- Backend: `3000`
- Frontend: `3001`
- MLflow: `5000`
- Redis: `6379`
- PostgreSQL: `5432`

## Python Development With uv

Use Python `3.12.x` and `uv` for Python modules such as `semd-backend` and `semd-ml`.

```bash
cd semd-backend
uv python install 3.12
uv sync
uv run pytest
uv run pyformat --in-place **/*.py
```

Run the same pattern inside `semd-ml` when working on the ML service.

## Frontend and Extension Development

Use Node.js commands inside the frontend or extension module that contains `package.json`.

```bash
cd semd-frontend
npm install
npm run dev
npm test
npm run build
```

For the browser extension, run the package scripts from `semd-extension`.

## Shared Local Network

Create the shared network before running containerized services:

```bash
podman network create semd-shared-network
```

## Useful Maintenance Commands

Kill running SEMD Python processes on Linux:

```bash
ps aux | grep semd | grep python | awk '{print $2}' | xargs kill -9
```
