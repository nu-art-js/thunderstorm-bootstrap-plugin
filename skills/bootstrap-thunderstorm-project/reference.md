# Reference — Bootstrap Thunderstorm Project (post-clone)

Assumes the repo was cloned from `thunderstorm-sample` (`nu-art-js/thunderstorm-sample`) **with `_thunderstorm` initialized** (`git clone --recurse-submodules` or `git submodule update --init --recursive`). Init BAI through the repo `build-and-install.sh` (it pins 0.500.x). Docker must be up before `bai -l` / e2e. Set `GCP_PROJECT_ID` to a real GCP project before password-auth.

## Product spec

When the user gives `specPath` (a local markdown file), read it with the Read tool, use it to fill gaps in the parameter table, then copy it into the new project as `_docs/specs/<sensible-name>.md`.

When `specPath` is empty, write `_docs/specs/product.md` as a pointer to Beamz knowledge for the project slug. The spec itself stays in Beamz (`https://api.beamz.dev/mcp/beamz`, server key = project slug). Do not invent one.

## GCP projects

Suggest creating `local`, `staging`, and `prod` in the parameter question. The first option is yes. Propose ids `nu-art-<short>-local|staging|prod`, each at most 30 characters, and show them before creating anything. Org `1056158311235`. Bill staging and prod to `012B92-EB17B2-394827`. Leave local unbilled.

Also suggest a staging-only deploy SA in `nu-art-dev-ops`. Create it with `bash scripts/create-staging-deploy-sa.sh`. **Do not put the JSON in the repo** — the script writes under `$HOME/.config/gcloud`. `gcloud builds submit` needs `roles/storage.admin` on `gs://nu-art-dev-ops_cloudbuild` and `roles/serviceusage.serviceUsageConsumer` on `nu-art-dev-ops`. Cloud Build runtime SA is `PROJECT_NUMBER-compute@developer.gserviceaccount.com`.

## Port map (template baseline N = 8000)

BAI reads literals in the app `__package.json` files, the backend listen fallback, and `app/e2e/src/test/sample-e2e-harness-constants.ts`. `bai-config.json` → `templateParams.params` mirrors those values and is not substituted. `mongo` is `port` and optional `dbName` only — do not add `dataDir`.

| Value | Formula | Sample |
|-------|---------|--------|
| debug / `PORT_BACKEND_DEBUG` | N | 8000 |
| frontend / `PORT_FRONTEND` | N + 1 | 8001 |
| backend / `PORT_BACKEND_APEX` | N + 2 | 8002 |
| config URL / `PORT_CONFIG` | N + 4 | 8004 |
| mongo / `PORT_MONGO` | 20000 + N | 28000 |
| E2E backend | N + 102 | 8102 |
| E2E mongo | 20000 + N + 21 | 28021 |

`FIREBASE_PROJECT_LOCAL`, `ARTIFACT_PROJECT_ID`, and `ARTIFACT_REGION` are the same kind of mirror. Live env ids are the `replace-dev` / `replace-staging` / `replace-prod` strings in the app `__package.json` files.

`node scripts/preflight-unit-config.mjs` checks the literals, the formulas, and the mirror against the installed `@nu-art/build-and-install`. Retarget the e2e constants file only, then rename `sample-e2e-*` to the project slug.

## Deploy (already in the clone)

`deploy.sh` + `deploy_rtdb_deltas.py` + `releases/`. Do not rewrite. Headless: drop `@app/frontend-vite` from `DEFAULT_UNITS`. Change unit names if the npm scope is not `@app`. Leave the prod gate (`DEPLOY_CONFIRM_PROD=yes`) in place.

## New capability package `__package.json` (typescript-lib)

Path: `<capability>/<domain>/__package.json`

```json
{
  "name": "<npmScope>/<capability>-<domain>",
  "version": "{{APP_VERSION}}",
  "private": true,
  "description": "<capability> — <DomainTitle>",
  "license": "<license>",
  "scripts": { "build": "tsc" },
  "publishConfig": { "directory": "dist", "linkDirectory": true },
  "dependencies": {
    "@nu-art/ts-common": "?"
  },
  "devDependencies": {
    "@types/chai": "?",
    "@types/mocha": "?"
  },
  "unitConfig": { "type": "typescript-lib" },
  "type": "module",
  "exports": {
    ".": { "types": "./index.d.ts", "import": "./index.js" },
    "./*": { "types": "./*.d.ts", "import": "./*.js" }
  }
}
```

Add domain deps (minimal):

| Domain | Add (typical) |
|--------|----------------|
| `shared` | `@nu-art/db-api-shared` |
| `frontend` (UI project) | `@<npmScope>/<capability>-shared`, `@<npmScope>/app-core-shared`, `@<npmScope>/app-core-frontend`, `@nu-art/thunder-core`, `@nu-art/db-api-frontend`, `@nu-art/db-api-shared`, `@nu-art/http-client` |
| `frontend` (before design spec accepted) | Add `app-core-*` deps (empty placeholder packages are fine); omit feature UI code until `_docs/specs/design-language.md` is filled by the human |
| `backend` | `@<npmScope>/<capability>-shared`, `@nu-art/db-api-backend`, `@nu-art/db-api-shared` |

Replace `<npmScope>` with the project scope (e.g. `@app`, `@quai`).

## `app-core` — design language implementation (UI projects)

Three-domain layout (same as template `core/`, renamed for clarity):

| Folder | Package name | Role |
|--------|----------------|------|
| `app-core/shared` | `@<npmScope>/app-core-shared` | Token types, semantic color/spacing keys, shared constants (no React). |
| `app-core/frontend` | `@<npmScope>/app-core-frontend` | Global styles, theme, primitives; **implements** `_docs/specs/design-language.md`. |
| `app-core/backend` | `@<npmScope>/app-core-backend` | Optional; server-driven theme or branding only. |

Use the same `__package.json` typescript-lib shape as other capabilities (see above). Wire `app-core-*` into the **host** frontend app (`app/frontend-vite`) like the template wires `core-*`.

## Design language spec — human-blocked stub

Create `_docs/specs/design-language.md` when the human has not supplied a doc yet. The agent must not invent the visual system; replace this stub after human review.

```markdown
# Design language — DRAFT (blocked on human)

**Status:** BLOCKED — human must complete before any feature UI work.

## Required sections (human fills all)

1. **Brand & tone** — personality, marketing vs. clinical density (if applicable).
2. **Typography** — font stacks, scale (e.g. modular steps), weights, line heights.
3. **Color** — semantic tokens (background, surface, text, border, primary, danger, success…), light/dark if applicable, contrast requirements (WCAG target).
4. **Spacing & layout** — base unit, scale, breakpoints, max content width.
5. **Shape** — border radius, dividers.
6. **Elevation** — shadows / layers.
7. **Motion** — duration/easing rules or “none”.
8. **Components** — which primitives exist in app-core vs. direct thunder-widgets usage.
9. **Thunderstorm integration** — how `@nu-art/thunder-widgets`, `@nu-art/ts-styles`, and routing shells align with the above.

## Agent implementation note

After this document is accepted, implement tokens in `app-core/shared` and visuals in `app-core/frontend` only; domain `*/frontend` packages consume app-core — no one-off global theme in domain packages.
```

## Minimal `src/main/index.ts` (library)

```typescript
export {};
```

Or a single named export placeholder.

## RTDB `configUrl` / `projectId`

Local pattern from template:

- `"projectId": "<your-local-firebase-project-id>"`
- `"configUrl": "http://127.0.0.1:<N+4>/_config/frontend/manager.json?ns=<your-local-firebase-project-id>-default-rtdb"`

## Removing template `core/` vs. adopting `app-core/`

**Headless repo (no kept frontend):** remove `core/` and strip `@app/core-*` from any remaining `app/backend` (or other) manifests; delete `core/*` directories.

**UI repo (`requiresUi`):** do not remove the design surface — **replace** template `core/` with **`app-core/`** (shared + frontend + optional backend), wire `@<npmScope>/app-core-*` into the host frontend (and backend if needed), then delete `core/*`. Remove old `@app/core-*` / `@<npmScope>/core-*` dependency keys and imports.

Dependencies to remove when dropping template `core/`:

- `@app/core-shared`, `@app/core-backend` from `app/backend/__package.json` (replace with `app-core-*` if used)
- `@app/core-frontend`, `@app/core-shared` from the kept frontend `__package.json` (replace with `app-core-*`)

Search `app/` for `@app/core-` / `@<npmScope>/core-` and fix imports and module packs.
