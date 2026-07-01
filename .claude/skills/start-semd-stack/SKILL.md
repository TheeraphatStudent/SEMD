---
name: start-semd-stack
description: >
  Bring up the full local SEMD stack (Postgres, Redis, MLflow, backend API, ML
  worker, frontend, and optionally the extension dev server) in the correct
  order. Trigger when the user asks to "start the stack", "run everything
  locally", "bring up backend and ml", or similar — not for starting a single
  service in isolation (just run that service's own dev command for that).
allowed-tools:
  - Bash
  - Read
  - Glob
---

## Steps

1. **Check prerequisites, don't assume them.**
   - Confirm the shared network exists: `podman network ls` (or `docker network ls`)
     for `semd-shared-network`. If missing, create it:
     `podman network create semd-shared-network`.
   - Confirm `.env` exists in `semd-backend/` and `semd-ml/`. If either is
     missing, check for `.env.example` in that directory and tell the user to
     `cp .env.example .env` and fill in real values — do not fabricate one.

2. **Start backend-side Docker services** (Postgres + Redis, via the include in
   `semd-backend/compose.yaml`) and the backend API:
   ```bash
   cd semd-backend
   docker compose up -d   # or: podman compose up -d
   ```

3. **Start the ML service + MLflow:**
   ```bash
   cd semd-ml
   docker compose up -d
   ```
   MLflow UI comes up at `http://localhost:5000`. If it fails on permissions,
   see `semd-ml/CLAUDE.md`'s MLflow permissions note before retrying.

4. **Start the frontend dev server** (not containerized in dev):
   ```bash
   cd semd-frontend && npm run dev   # localhost:3000 (Next.js) — note backend
                                      # itself listens on 3000/8000 per README;
                                      # confirm no port clash before starting both
   ```

5. **Report status**, not just "done" — run `docker ps` / `podman ps` and show
   which containers are actually healthy vs. still starting, and which ports
   are bound. If any container exited immediately, surface its logs
   (`docker logs <container>` / `podman logs <container>`) rather than
   declaring success.

6. Do not start the browser extension dev server automatically — it's a
   separate concern (`build-extension` skill) and isn't part of "the stack."

## Notes

- Never run `docker compose down -v` / `podman compose down -v` as part of
  this skill — that destroys Postgres/Redis volumes. Bringing the stack up
  should never imply tearing existing data down first.
- If a compose command fails because the network doesn't exist yet, that's
  the prerequisite check in step 1 failing silently — re-verify, don't retry
  blindly.
