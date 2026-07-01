---
name: sync-api-client
description: >
  Regenerate semd-frontend's typed API client after semd-backend's Pydantic
  models, routers, or openapi.yaml change. Trigger after editing anything in
  semd-backend/models/, semd-backend/routers/, or semd-backend/openapi.yaml,
  or when the user explicitly asks to "regenerate the API client" / "sync the
  frontend types". Do not trigger for unrelated frontend-only or ML-only
  changes.
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
---

## Steps

1. **Confirm there's actually a backend contract change to sync.** Check
   `git status`/`git diff` inside `semd-backend/` for changes under `models/`,
   `routers/`, or to `openapi.yaml` itself. If none, say so and stop — don't
   regenerate for no reason.

2. **Regenerate the client**, from `semd-frontend/`:
   ```bash
   cd semd-frontend
   npm run generate:api
   ```
   This runs `orval` against the config in `semd-frontend/orval.config.ts`,
   writing into `semd-frontend/src/services/generated/`.

3. **Diff the generated output** (`git diff` inside `semd-frontend/`) and
   summarize what changed — new endpoints, changed request/response shapes,
   renamed types — rather than just reporting "regenerated successfully".

4. **Check for consumers that now break.** Grep `semd-frontend/src` for usages
   of any type/function whose shape changed in step 3, and flag call sites
   that likely need updating. Do not silently edit call sites — report them.

5. Remember `semd-backend` and `semd-frontend` are separate git repos: the
   regenerated client is a `semd-frontend`-only change and must be committed
   there, independent of whatever commit changed the backend contract.

## Notes

- Never hand-edit files under `src/services/generated/` — they're
  overwritten on the next `generate:api` run. If a generated type looks
  wrong, the fix belongs in the backend's Pydantic model or `openapi.yaml`,
  not in the generated file.
- If `npm run generate:api` fails, check that the backend's `openapi.yaml` is
  actually current (regenerate it from the running FastAPI app at
  `/openapi.json` if the backend has since changed) before assuming orval or
  the config is broken.
