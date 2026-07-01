# Reorg Audit Findings (2026-07-01)

## Verdict: no reorganization warranted

A graphify code-graph audit (`graphify-out/GRAPH_REPORT.md`) was proposed as the basis for
a project-structure reorg. After verification, the premise doesn't hold: every specific
number cited turned out to be unreconcilable with the actual repo state, and every flagged
"god node" is legitimate architecture rather than over-coupling.

## What was claimed vs. what's actually true

| Claim | Verified reality |
|---|---|
| 535 dangling-endpoint edges, 120 collapsed edge pairs | Not recorded in any persisted artifact. Direct recount on `graphify-out/graph.json` (the graph actually used for analysis) shows **0** dangling edges, **0** collapsed pairs, **0** self-loops. |
| 5 sensitive files skipped | Only **3** files match any sensitive-filename pattern, all `.env.example`/`.example.env` templates: `semd-frontend/.env.example`, `semd-frontend/.example.env`, `semd-ml/src/.env.example`. No 4th/5th file exists anywhere in the repo. |
| 25 docs / 27 images uncovered, needing placement | Inventoried by subagent — all docs and images already sit in the correct domain directory. No cross-domain misplacement found. |
| 10 "god nodes" signal over-coupling | All 10 resolved to exactly one source node each (no name-collision merges). Every one is legitimate: shared base classes, core pipeline stages, or shared UI primitives — high fan-in is expected, not a smell. |
| "Surprising" cross-domain connections | `ImportVerifier → RedisClient/PredictionService/DatasetPipeline`: artifact of `semd-ml/src/verify_imports.py`, a smoke-test script that imports every module to check nothing's broken — not real architectural coupling. `ReportRoute → ReportModelResponse/ReportModelRequest`: real, but an ordinary same-package import inside `semd-backend`, not actually cross-domain or surprising. |

## God node classification

| Node | Location | Verdict |
|---|---|---|
| `BaseResponseModel` | `semd-backend/models/base_response_model.py` | Legit Pydantic base class |
| `BaseRoute` | `semd-backend/routers/base_route.py` | Legit FastAPI router base class |
| `FeatureExtractor` | `semd-ml/src/features/feature_extractor.py` | Legit core ML pipeline stage |
| `MLPipeline` | `semd-ml/src/ml/ml_pipeline.py` | Legit core ML pipeline stage |
| `DatasetPipeline` | `semd-ml/src/data/dataset_pipeline.py` | Legit core ML pipeline stage |
| `ApiClient` | `semd-frontend/src/lib/api-client.ts` | Legit shared HTTP client singleton |
| `Card` / `CardHeader` / `CardContent` | `semd-frontend/src/components/ui/card.tsx` | Legit shared UI primitives, one file |
| `ImportVerifier` | `semd-ml/src/verify_imports.py` | Artifact — smoke-test script, not architectural coupling |

## Structural context

The repo is already domain-split at the top level: `semd-ml/`, `semd-backend/`,
`semd-frontend/`, `semd-extension/`. Each of these four directories is in fact a separate
git submodule pointing at its own GitHub remote
(`TheeraphatStudent/SEMD-{ml,backend,frontend,extension}`). "Reorganize the project
structure" has very little to actually do given this is already the structure.

## Items noted but out of scope for this pass

- `semd-extension/chrome_extension/`, `firefox_addons/`, and `out/` (37 files) are
  committed build output tracked inside the `semd-extension` submodule, sitting next to
  source. Worth a `.gitignore` + untrack pass, but the submodule is currently in a
  detached-HEAD state on a separate remote and the parent repo has unrelated staged
  changes in flight — deferred rather than touched in this pass.
- `semd-frontend/design/*.png|jpg` holds UI mockup references outside the `public/`
  asset convention used elsewhere in that domain. Not wrong, just inconsistent — worth a
  convention decision later.

## Bottom line

No files were moved. No code was changed. The graph audit's numeric claims don't survive
verification against the actual repo, and the god nodes it flagged are healthy
architecture. Recommend closing this exercise without a reorg, and separately deciding
whether to clean up the two noted build-output/asset-convention items when the submodule
migration is in a stable state.
