---
name: bootstrap-thunderstorm-project
description: >-
  Scaffold a new Thunderstorm monorepo by cloning the official sample boilerplate,
  then renaming, re-pointing git, adjusting ports and Firebase project IDs, and
  optionally adding capability packages. UI projects require a human-authored design
  language spec before feature UI; agent implements app-core/shared and app-core/frontend.
  Accepts optional product and design-language spec paths. Use for "new thunderstorm
  project", "bootstrap BAI monorepo", or "clone thunderstorm template".
---

# Bootstrap Thunderstorm Project (clone-based)

Do **not** hand-author the full monorepo from scratch. Start from the maintained boilerplate, then customize.

**Source of truth:** `git@github.com:nu-art-js/thunderstorm-sample.git` (Thunderstorm 0.500.x, Vite frontend, sample `core/` library, BAI, `_thunderstorm` submodule).

After clone, read `_thunderstorm/.rules/operational/bai-cli.mdc` and `_thunderstorm/.rules/operational/project-structure.mdc` before running BAI.

## Step 1 — Collect ALL parameters in ONE round

**Collect every parameter in a single `AskQuestion` call.** Do not split across multiple rounds. If a parameter has a sensible default, show it as the first option. When `requiresUi` is true, default every capability concept to all three domains (`shared` + `backend` + `frontend`) — do not propose a minimal set and wait for correction.

| Parameter | Notes |
|-----------|--------|
| `projectName` | Directory name and human label |
| `npmScope` | The `@org` prefix only (e.g. `@app`), not `@org/pkg`. Must match how you name workspace packages |
| `gitRepoUrl` | New repo URL (`git@github.com:org/repo.git`) |
| `templateRepoUrl` | Default: `git@github.com:nu-art-js/thunderstorm-sample.git` |
| `thunderstormCheckout` | Optional: tag or branch for `_thunderstorm` after clone (default: keep template’s submodule pointer) |
| `author`, `license` | For `__package.json` / package metadata where applicable |
| `firebaseProjectIds` | Suggest `local`, `staging`, and `prod`. Do not add `dev` unless they pick it. Propose ids in the question (see step 6b). |
| `createGcpProjects` | **Suggest yes.** First option: create those projects under the nu-art org and enable Firebase. Show the exact ids and display names. |
| `port` | Integer **N** — base for local ports (see [reference.md](reference.md)) |
| `keepFrontend` | Default true. Template frontend is **Vite only** (`app/frontend-vite`). If false (headless), delete that tree. |
| `initialPackages` | e.g. `["messaging/shared","messaging/backend","messaging/frontend"]` — new capability folders |
| `removeSampleCore` | If true: delete template `core/` and strip `@app/core-*` deps — **see Design language gate** if the project keeps a frontend |
| `specPath` | **Optional.** Absolute path or workspace-relative path to a product/architecture spec (markdown). User may paste or `@`-reference it. Read it **before** clone/customize to infer `projectName`, `initialPackages`, `keepFrontend`, phases, and integration notes. After bootstrap, optionally **copy** the spec into the new repo under `_docs/specs/` so the codebase carries its own contract. |
| `requiresUi` | **Implicitly true** when you keep `app/frontend-vite`. When true, a **design language spec** is mandatory before feature UI work; see Design language gate. |
| `designLanguageSpecPath` | **Optional.** Path to an existing design-language doc to copy into the new repo. If `requiresUi` and this is empty, **do not** implement screens or visual domain UI in bootstrap — create `_docs/specs/design-language.md` as a **human-blocked stub** (see reference.md). |

If `specPath` is set and `initialPackages` is empty, derive capability folders from the spec (sections, resource names, or explicit package list in the doc). If both conflict, **ask the user** which wins.

### Product spec lives in Beamz

When `specPath` is empty, the product spec is not a file in the new repo. It lives in **Beamz knowledge** for the MCP server key from step 5b (`https://api.beamz.dev/mcp/beamz`).

- Do not invent a product spec, and do not block the clone because one is missing.
- Write `_docs/specs/product.md` as a pointer only: the spec is Beamz knowledge for `<projectName>`. Do not paste a guessed spec there.
- Later work reads the spec from that Beamz project. Copy it into the repo only when the human asks for a snapshot.
- A local `specPath` still wins when the human hands you a file: read it, derive packages, and copy it under `_docs/specs/` as today.

## Design language gate (projects with UI)

Applies whenever the monorepo keeps a Thunderstorm frontend app (`vite-hosting`).

### Ownership

| Who | Responsibility |
|-----|----------------|
| **Human** | Authors the **design language spec** — typography, color tokens (semantic + scale), spacing/radius, elevation, density, motion (if any), accessibility/contrast rules, and how the app extends Thunderstorm / `@nu-art/thunder-widgets` / `@nu-art/ts-styles`. |
| **Agent** | After the spec exists (or alongside minimal bootstrap), implements it as **`app-core`** packages — same **three-domain layout as template `core/`**, but named `app-core/shared`, `app-core/frontend`, and `app-core/backend` only if server-side theme/config is needed. The **visual system** lives primarily in **`app-core/frontend`** (global styles, theme provider, primitives). |

### Hard rule

- **No feature UI implementation** (routes, pages, domain-specific components with real layout/visual design) until `_docs/specs/design-language.md` (or `designLanguageSpecPath`) is **filled and accepted** by the human.
- Bootstrap may still: clone, wire BAI, add **empty** capability packages, backend APIs, and **non-visual** plumbing.
- If the human has not provided content yet: create `_docs/specs/design-language.md` from the **stub** in [reference.md](reference.md), commit it, and treat all UI tasks as **blocked** until the human replaces the stub.

### Package layout (`app-core`)

- **`app-core/shared`** — **type-only vocabulary**. String-literal unions naming variants (`ButtonVariant`, `StatusVariant`, `TypeScale`, `SpacingToken`…). **No values.** No hex codes, no px, no font sizes, no spacing numbers, no font weights. Just names. No React.
- **`app-core/frontend`** — owns **every** style value. SCSS partials defining palette, `:root` CSS variables (the single source of runtime values), and class-based primitives (`.btn-primary`, `.type-body`, `.card-raised`…). Optional thin React wrappers that accept `app-core-shared` types and apply class names.
- **`app-core/backend`** — optional; use only if the design contract includes server-driven branding or theme payloads.

### Hard rule: SCSS-only styling

- **No style values in TypeScript.** Hex codes, px values, font sizes, spacing numbers, line heights, easing curves, shadows — all live exclusively in SCSS files inside `app-core/frontend`.
- **No inline `style={{...}}`** in JSX, except for genuinely dynamic computed values that cannot be expressed as a class (e.g. a progress bar's percentage width). "Looks easier inline" is not a reason.
- **No CSS-in-JS** (styled-components, emotion, etc.). Class names + SCSS only.
- **No re-declaring tokens** in domain packages. If a value is needed, it comes from a `:root` CSS variable defined in `app-core/frontend/styles/_tokens.scss`. If it's missing there, add it there — not in the consumer.
- TypeScript may **reference** variant names via the type unions in `app-core/shared`; SCSS rules turn those names into visuals.

Wire **`@<npmScope>/app-core-shared`** and **`@<npmScope>/app-core-frontend`** into the **kept app frontend** `__package.json`. Domain `*/frontend` packages should depend on **`app-core-frontend`** (and shared) so they **compose** the design system instead of inventing ad-hoc styles.

### `app-core` is mandatory when `requiresUi` — regardless of `removeSampleCore`

When `requiresUi` is true, the **`app-core/`** packages (`shared` + `frontend`, optionally `backend`) **always** get created during bootstrap. `removeSampleCore` only controls whether the old template `core/` is deleted — it does **not** control whether `app-core/` exists.

| `removeSampleCore` | `requiresUi` | Outcome |
|--------------------|--------------|---------|
| `true` | `true` | Delete `core/`, create `app-core/`, wire `app-core-*` into host frontend and domain frontends. |
| `false` | `true` | Keep `core/` (human may refactor later), **and also** create `app-core/` alongside it. Domain frontends depend on `app-core-*`, not `core-*`. |
| `true` | `false` | Delete `core/`, skip `app-core/` (headless). |
| `false` | `false` | Keep `core/` as-is (API-only, no UI concern). |

### Adding a frontend later (headless-to-UI transition)

If a repo was bootstrapped without a frontend and a frontend app is added later, the design language gate activates at that point: create the `_docs/specs/design-language.md` stub, add `app-core/` packages, and block feature UI until the human fills the spec.

### Task ordering (for agents planning work)

1. Product/architecture spec (optional `specPath`).
2. **Design language spec** (human) → `_docs/specs/design-language.md`.
3. **Agent** scaffolds or implements **`app-core`** from that doc.
4. Domain capability UI (`*/frontend`) and app shell routes.

### Lazy design language handoff

During bootstrap the agent creates `app-core/` as **empty placeholders** and commits the stub spec. When the human later replaces the stub with a real `_docs/specs/design-language.md`:

1. **Read** the completed spec.
2. **Define type vocabulary** in `app-core/shared/src/main/types.ts` — string-literal unions for variants only, no values.
3. **Implement values** in `app-core/frontend/src/main/styles/` — palette partial, `:root` tokens, and class-based primitives in SCSS. Every hex/px/weight value goes here and only here.
4. Build with BAI and commit.

Only **after** step 4 is feature UI (routes, pages, domain layout) unblocked.

### Ongoing: new `*/frontend` packages must depend on `app-core`

This rule applies **at all times**, not just during bootstrap. Whenever a new `*/frontend` capability package is created in a `requiresUi` repo, it **must** list `@<npmScope>/app-core-frontend` and `@<npmScope>/app-core-shared` in its `__package.json` dependencies. Domain frontends that skip `app-core` and invent ad-hoc global styles violate the design contract.

## Step 2 — Clone and init the Thunderstorm submodule

```bash
git clone --recurse-submodules <templateRepoUrl> <projectName>
cd <projectName>
```

If the clone was not recursive, **immediately**:

```bash
git submodule update --init --recursive
```

`_thunderstorm` is a git submodule. An empty `_thunderstorm/` cannot compile. Do not run BAI until `ls _thunderstorm/e2e-harness/__package.json` (or any other package `__package.json`) succeeds.

The template pointer tracks thunderstorm `origin/main`. BAI is the latest published `@nu-art/build-and-install`. `version-thunderstorm.json` must name that version (`npm view @nu-art/build-and-install version`). Do not pin an older BAI, and do not leave the pointer behind `main`.

Optional: `cd _thunderstorm && git fetch && git checkout <thunderstormCheckout> && cd ..` then commit the submodule pointer if you changed it.

## Step 3 — Point git at the new remote

```bash
git remote set-url origin <gitRepoUrl>
git remote remove template 2>/dev/null || true
```

A leftover `template` remote (from renaming `origin` → `template` before adding the product remote) must be removed. Do not leave the sample repo as a second remote.

Keep history from the sample, or `rm -rf .git && git init` if you want a clean history (then add remote and initial commit).

## Step 4 — Frontend is Vite only

The template ships **`app/frontend-vite`**. There is no webpack app.

- **`keepFrontend` true** (default): keep `app/frontend-vite/`.
- **Headless:** delete `app/frontend-vite/` (including `__package.json`). Do not leave an orphan package in the workspace.

## Step 5 — Rename workspace root package

In root `__package.json`:

- Set `"name"` from `@app/sample-project` to `<npmScope>/<projectName>` (or `<npmScope>/<slug>` if npm name must differ from folder).

Regenerate workspace metadata with BAI after edits (see Step 9).

## Step 5b — Beamz MCP project

`.cursor/mcp.json` is already in the clone. The server key is how Beamz knows which project this checkout is.

- Replace `<project-name>` with the project slug.
- Leave the URL `https://api.beamz.dev/mcp/beamz`.
- Do not point it at a local host, and do not add other MCP servers during bootstrap.

## Step 6 — Firebase project IDs and RTDB config URL

The template already has `local` / `dev` / `staging` / `prod` on the backend and the Vite app. Do not add a fifth env or a second config file.

BAI reads literals. `bai-config.json` `templateParams.params` is a mirror: update it in the same edit, and do not expect `{{PARAM}}` to be substituted.

- Write `firebaseProjectIds.local` as a literal on the backend and frontend `local.projectId`, and in the local `ns=` hostname. Copy the same id into `FIREBASE_PROJECT_LOCAL`.
- Replace `replace-dev`, `replace-staging`, and `replace-prod` (project ids and the matching `*-default-rtdb` hostnames) with the ids the user supplied. Leave a `replace-*` placeholder only when the user did not give that env.
- Write the artifact project and region as literals on `containerDeployment` and `hostingDeployment`. Copy them into `ARTIFACT_PROJECT_ID` and `ARTIFACT_REGION`. Set `imageName` and `packageName` from the project slug (`<slug>-backend`, `<slug>-frontend`). Repository names `web-apps` and `hosting-builds` stay unless the user says otherwise.

## Step 6b — Suggest GCP projects, then create them

In the step 1 question, suggest creating the projects. The first option is yes. Show the exact ids and display names. Default envs are `local`, `staging`, and `prod`. Do not add `dev` unless they pick it.

Ids and display names are each at most 30 characters. Propose `nu-art-<short>-local`, `nu-art-<short>-staging`, and `nu-art-<short>-prod`. If the project name does not fit, shorten it in the question before they answer. `"Business Identity Syncer staging"` is 34 characters and is rejected.

If they accept:

- Create the projects under the nu-art org (`1056158311235`, `nu-art-software.com`) and enable Firebase on each.
- Link staging and prod to billing account `012B92-EB17B2-394827` (Firebase Payment). Leave the `*-local` project unbilled.
- Write those ids into the app `__package.json` files. Do not leave `replace-*` for an env you created.
- If the user `gcloud` token cannot refresh, call the APIs with application-default credentials. Do not run `gcloud config set`.

```bash
export CLOUDSDK_AUTH_ACCESS_TOKEN="$(gcloud auth application-default print-access-token)"
export CLOUDSDK_CORE_DISABLE_PROMPTS=1
```

If they decline, leave `replace-*` only for envs they did not supply.

## Step 6c — Suggest a staging deploy service account

In the same parameter round, suggest creating a **staging-only** deploy SA in `nu-art-dev-ops`. First option is yes. The account and its JSON key stay in GCP and in a Cursor secret. **Never commit a key, never paste JSON into the repo, chat, or Beamz.**

Name: `<slug>-staging-deploy@nu-art-dev-ops.iam.gserviceaccount.com`. No Owner, Editor, or prod.

Enable on `nu-art-dev-ops`: `cloudbuild.googleapis.com`, `artifactregistry.googleapis.com`, `cloudresourcemanager.googleapis.com`. Enable on the staging project: `run`, `firebase`, `firebasehosting`, `firebasedatabase`. Confirm `web-apps` and `hosting-builds` already exist; do not create them with this SA.

Grants that `gcloud builds submit` actually needs:

- `roles/serviceusage.serviceUsageConsumer` on `nu-art-dev-ops`
- `roles/cloudbuild.builds.editor` on `nu-art-dev-ops`
- **`roles/storage.admin` on `gs://nu-art-dev-ops_cloudbuild`** — editor plus `storage.objectAdmin` is not enough
- `roles/artifactregistry.writer` on `web-apps` and `hosting-builds` only
- `roles/iam.serviceAccountUser` on the Cloud Build **runtime** SA: `PROJECT_NUMBER-compute@developer.gserviceaccount.com` (the legacy `PROJECT_NUMBER@cloudbuild.gserviceaccount.com` often does not exist)

On staging: `run.developer`, `firebasehosting.admin`, `firebasedatabase.admin`, `serviceUsageConsumer`, `serviceAccountUser` on the default compute SA.

Robots: Cloud Build runtime SA writer on `web-apps`; staging Cloud Run agent reader on `web-apps`.

Run this skill's `create-staging-deploy-sa.sh` from the **product** repo (the script lives next to this SKILL.md, not in the clone):

```bash
bash ~/.cursor/skills/bootstrap-thunderstorm-project/create-staging-deploy-sa.sh <slug> <staging-project-id>
```

It writes `$HOME/.config/gcloud/<slug>-staging-deploy.json` and refuses if that path is inside the product worktree. Never `cat` the file. Do **not** copy the script into the new repo — the next skill update would never reach old clones. Put `*-staging-deploy.json` in the product `.gitignore`.

Cloud Agents do not inherit the laptop's SDK or ADC. For a product that already has a Cloud install/start hook: init `_thunderstorm` in **start** (`git -c url.https://github.com/.insteadOf=git@github.com: submodule update --init --recursive`) so the agent does not clone it mid-prompt; in **install** put host tools (`cpio`, `rsync`), the Cloud SDK (`$HOME/google-cloud-sdk`), and `docker.io`. Start the docker daemon in **start** (`service docker start`) — the snapshot does not keep it running. **Never** run `build-and-install.sh init` in `install` — it OOMs the Cloud Build. Agents run BAI when they actually need it. Do not copy a Dockerfile from the sample. The human must add a Cursor secret named **`<SLUG>_STAGING_DEPLOY_SA_JSON`** (hyphens → underscores, upper case; Identity: `IDENTITY_SYNCER_STAGING_DEPLOY_SA_JSON`) with the key file contents, not a path. Do not invent a second generic secret name. `deploy.sh` reads that env and `gcloud auth activate-service-account`. Local Docker is not required for `deploy.sh` (Cloud Build builds the image); Cloud Agents still need Docker for `bai -l` / e2e.

Do not copy a Cloud Dockerfile / bootstrap.sh from the sample into the product. If they need `cpio`/`rsync`/`google-cloud-cli` on a Cloud agent, add those packages to **that product's existing** install hook. Host-tool lists belong here so they can change without freezing a Dockerfile in every app.

## Step 7 — Ports

BAI reads these literals: backend `debugPort`, `basePort`, and `mongo.port`; frontend `servingPort` and the config URL; the listen fallback in `app/backend/src/main/index.ts`; the e2e constants file. `mongo` is `port` plus optional `dbName`. Do not add `dataDir`.

`bai-config.json` `templateParams.params` must carry the same numbers. Published BAI 0.500.6 does not substitute them. `{{APP_VERSION}}` still works; it comes from `templateParams.packageJson`.

Base port is N. Do not invent offsets.

| Value | Formula | Sample (N = 8000) |
|-------|---------|-------------------|
| `PORT_BACKEND_DEBUG` / `debugPort` | N | 8000 |
| `PORT_FRONTEND` / `servingPort` | N + 1 | 8001 |
| `PORT_BACKEND_APEX` / `basePort` / listen fallback | N + 2 | 8002 |
| `PORT_CONFIG` (inside the local config URL) | N + 4 | 8004 |
| `PORT_MONGO` / `mongo.port` | 20000 + N | 28000 |
| E2E backend | N + 102 | 8102 |
| E2E mongo | 20000 + N + 21 | 28021 |

Keep **`app/e2e`**. It is a product consumer of `@nu-art/e2e-harness`. Do not add `app/e2e-harness` or a second stack. Retarget only `app/e2e/src/test/sample-e2e-harness-constants.ts` using the formulas above (backend **N+102**, mongo **20000+N+21**, Firebase project id = `firebaseProjectIds.local`).

Rename the `sample-e2e-*` filenames, the `SAMPLE_E2E_*` constants, and the mongo container name `mongo-emu-sample-e2e-harness` to the project slug. Do not leave the word `sample` in the e2e package.

Also search for `localhost:8008` (storage emulator proxy in backend) only if you intentionally change storage proxy wiring — default leave as-is unless documented otherwise.

## Step 7b — Deploy script (already in the clone)

`deploy.sh`, `deploy_rtdb_deltas.py`, `deploy_rtdb_deltas_test.py`, and `releases/README.md` ship with the template. Do not rewrite them and do not add a second deploy script.

- Headless (`keepFrontend` false): remove `@app/frontend-vite` from `DEFAULT_UNITS` in `deploy.sh`.
- Otherwise leave `DEFAULT_UNITS` as `@app/backend,@app/frontend-vite`. If the npm scope is not `@app`, change those unit names to match.
- `BACKEND_UNIT` must stay the package whose `unitConfig.containerDeployment` the delta walker reads (`app/backend/package.json`).
- Prod stays gated on `DEPLOY_CONFIRM_PROD=yes`. Do not remove that gate.

## Step 8 — Metadata and docs

- Update first line of `CLAUDE.md` to `# <projectName>` (body already points at `_thunderstorm` rules).
- Set `author` / `license` in root `__package.json` if collected.
- Update `repository.url` in app `__package.json` files to `<gitRepoUrl>` (optional but cleaner).
- `_docs/` already exists in the template; add project-specific content as needed.
- If **`requiresUi`**: add `_docs/specs/design-language.md` — either copy from `designLanguageSpecPath` or create the **human-blocked stub** from [reference.md](reference.md). Do not implement visual feature UI in the same change as bootstrap unless the human has already supplied a complete design language doc.

## Step 9 — `app-core/` and sample `core/`

### 9a — Create `app-core/` (always when `requiresUi`)

If **`requiresUi`** is true, add **`app-core/shared`**, **`app-core/frontend`**, and optionally **`app-core/backend`** per [reference.md](reference.md) naming (`@<npmScope>/app-core-*`). Wire `app-core-*` into the kept frontend (and backend if needed). This happens **regardless** of `removeSampleCore`.

### 9b — Remove template `core/` (only when `removeSampleCore` is true)

1. Delete `core/` (all three domains).
2. Remove `@app/core-shared` and `@app/core-backend` from `app/backend/__package.json`; replace with `@<npmScope>/app-core-shared` / `app-core-backend` when they exist.
3. Remove `@app/core-frontend` / `@app/core-shared` from the kept frontend `__package.json`; the `app-core-*` deps from 9a already fill those slots when `requiresUi`.
4. Search `app/` for remaining `@app/core-` references and fix imports / module packs.

If `removeSampleCore` is true and **`requiresUi`** is false (headless / API-only repo with no kept frontend), skip `app-core/` and only strip `core-*` deps that applied to removed apps.

## Step 10 — Add `initialPackages`

For each path like `messaging/shared`:

1. Create `messaging/shared/`, `messaging/frontend/`, etc. as needed.
2. Add `__package.json` per [reference.md](reference.md) (typescript-lib pattern, `{{APP_VERSION}}`, exports).
3. Add minimal `src/main/index.ts` (and `src/test/` if tests expected).
4. Wire the new packages into the **kept** app `__package.json` dependencies so BAI discovers them in the graph.
5. If **`requiresUi`** and any new package is `*/frontend`, add **`@<npmScope>/app-core-frontend`** and **`@<npmScope>/app-core-shared`** to that package’s `__package.json` dependencies (design system composition — not optional for domain UI).

## Step 11 — Dev workspace launcher config

The clone already has `.cursor/dev-workspaces/workspaces.json` and `.cursor/project-config.yaml`. `.cursor/scratch/` is gitignored. Do not add a second manifest or a per-project launcher script.

Rename the `sample-dev` key and title to `<projectName>-dev`. Point pane `-up=` values at the kept package names (`@app/backend` and `@app/frontend-vite`, or the new scope if you renamed `@app` everywhere). Drop the frontend pane when headless. Keep the `sleep` stagger.

The `launch-workspace` skill must be installed machine-wide (`~/.cursor/skills/`) for this to run.

## Step 12 — Initial setup

From project root, run a **full** workspace init **through the repo script**:

```bash
bash build-and-install.sh init
```

This is the **only** correct bootstrap command for a fresh clone — it creates a temporary `package.json`, installs tsx via pnpm, then runs the full BAI pipeline. **Do not** run `initial-setup.sh` or pass raw flags like `-fs -th -p -cox` — those assume tsx is already installed and will fail on a fresh clone.

The sample’s `build-and-install.sh` injects `--ts-version` from `version-thunderstorm.json` (0.500.x). The **upstream** BAI wrapper still defaults to `~0.401.0` and does **not** read that file.

**Never** `bai -i -up=<subset>` (or `bash build-and-install.sh -i -up=…`). A subset install rewrites a broken `pnpm-workspace.yaml` and drops the rest of the monorepo. Use full-workspace `init` or `bai -i -nb` only.

Never use raw `pnpm install` / `pnpm run build` as the primary workflow — BAI owns the lifecycle.

### Preflight (once BAI is installed)

`build-and-install.sh` runs `node scripts/preflight-unit-config.mjs` before later BAI commands. The first `init` on a fresh clone skips it, because `node_modules` does not exist yet. After that, the script checks every `__package.json` `unitConfig` against the **installed** `@nu-art/build-and-install`, and checks that the port formulas and the `bai-config.json` mirror match the literals.

If `init` fails on `unitConfig` or `Missing template param`:

1. Run `node scripts/preflight-unit-config.mjs`.
2. Diff the installed `UnitMapper_*.js` validator with the matching file under `_thunderstorm`.
3. Change the template to satisfy the installed package. Do not rewrite this skill from the first error line. The submodule can accept `mongo.dataDir` and `{{PARAM}}` while published 0.500.6 rejects both.

### Verify BAI is 0.500.x (mandatory)

After init (and after any later `bai -i`):

```bash
node -p "require('./node_modules/@nu-art/build-and-install/package.json').version"
```

Must match `^0\.500\.`. Also check the init log line `Resolved TS_VERSION from npm registry:` / `TS_DESIRED_VERSION set from CLI flag`.

**If you see 0.401.x:** stop. Do not compile. You bypassed the project wrapper (cached `bundle.bai.sh` invoked without the script, `pnpm exec build-and-install`, or a copied `build-and-install.sh` that lost the pin). Tell the user, then re-init with the hack:

```bash
PIN="$(python3 -c "import json; print(json.load(open('version-thunderstorm.json'))['version'])")"
bash build-and-install.sh init --ts-version="$PIN"
# equivalent: TS_VERSION="$PIN" bash build-and-install.sh init
```

### Docker (required to run)

`bai -l` (backend) and `bai -t -tt=pure -up=@app/e2e$` start Docker Mongo + Firebase emulators.

```bash
docker info   # must succeed; if not, start Docker Desktop and retry
```

Do not treat emulator/mongo failures as app bugs until Docker is up.

### Host tools: `cpio` and `rsync`

BAI shells out to both. `cpio` copies SCSS and other assets into each package `dist`. `rsync` copies dependency output into the backend container tree. Install them on every machine and cloud image before the first build:

```bash
# Debian/Ubuntu, including Cursor Cloud `/workspace`
apt-get install -y cpio rsync
```

macOS already ships both. A missing `cpio` is hidden: the copy command drops stderr. `dist` then has no `index.scss`, and the Vite failure looks like a bad `@nu-art/ts-styles` package entry.

### GCP / JWT (required to register/login)

Session JWT uses Secret Manager. Before launch or e2e:

```bash
export GCP_PROJECT_ID=<real-gcp-project>   # not demo-project / *-local
```

Or `gcloud config set project <id>` with working ADC. `GCLOUD_PROJECT` / `GOOGLE_CLOUD_PROJECT` stay on the emulator id.

### E2E mocha children (keep these when editing `@app/e2e`)

- Spawned `firebase` CLI and `node dist/index.js` must **delete `NODE_OPTIONS`**. BAI ts-mocha registers `ts-node/esm` and that crashes those children.
- Do not pass `FIREBASE_CONFIG` to the Firebase CLI.
- Keep the GCP / emulator project split above.

## Step 13 — Commit

Commit with a clear message (e.g. `bootstrap: <projectName> from thunderstorm-sample`).

## Key rules

- **Clone first** — boilerplate lives in `thunderstorm-sample`; the skill customizes, it does not recreate the tree file-by-file.
- **Init the `_thunderstorm` submodule** before any BAI command (`git clone --recurse-submodules` or `git submodule update --init --recursive`).
- **Pin Thunderstorm to latest** — BAI version is `version-thunderstorm.json` (and `bai-config.json` `THUNDERSTORM_VERSION`), and it must match `npm view @nu-art/build-and-install version`. The `_thunderstorm` pointer tracks `origin/main`. The sample `build-and-install.sh` injects `--ts-version` from that file. **Verify** `node_modules/@nu-art/build-and-install` matches the pin after init. If it is 0.401.x, you bypassed the project wrapper. Re-init with `--ts-version=<pin>`.
- **Docker must be running** before `bai -l` or product e2e. **`GCP_PROJECT_ID`** must be a real GCP project before password-auth / JWT.
- **Host tools:** install `cpio` and `rsync` before BAI. `cpio` copies assets into `dist`. `rsync` copies dependency output for the backend image. A missing `cpio` fails silently and the Vite build then cannot resolve `@nu-art/ts-styles`.
- **Never edit BAI-generated** `package.json` / `pnpm-workspace.yaml` — only `__package.json` templates and source; run BAI to regenerate.
- **Never subset-install** — `bai -i -up=<subset>` rewrites a broken workspace file. Full `init` / `bai -i -nb` only.
- **Keep `@app/e2e`** — retarget the constants file and rename `sample-e2e-*` to the project slug. Do not add another `@app/e2e-harness` copy (use `@nu-art/e2e-harness`). Do not add Jest or Vitest. Firebase tests are `*.test.firebase.ts` via `stormTester`; Playwright tests are `*.test.playwright.ts`.
- **Ports and project ids:** BAI reads literals in the app `__package.json` files, `app/backend/src/main/index.ts`, and the e2e constants. Mirror those values in `bai-config.json` `templateParams.params`. Formulas, with base N: debug N, frontend N+1, backend N+2, config N+4, mongo `20000+N`, e2e backend N+102, e2e mongo `20000+N+21`. Published BAI 0.500.6 does not substitute `templateParams.params`. `mongo` is `port` plus optional `dbName` only. `node scripts/preflight-unit-config.mjs` enforces this against the installed package.
- **Beamz MCP:** replace `<project-name>` in `.cursor/mcp.json` with the project slug. Leave `https://api.beamz.dev/mcp/beamz`. The product spec lives in that Beamz knowledge project. Without `specPath`, write only a pointer at `_docs/specs/product.md`.
- **Do not rewrite `deploy.sh` or `deploy_rtdb_deltas.py`.** Retarget `DEFAULT_UNITS` / image names / `replace-*` project ids only. Config deltas are `releases/<semver>.json`. Prod stays behind `DEPLOY_CONFIRM_PROD=yes`.
- **`?` and `{{APP_VERSION}}`** — keep template conventions; versions resolve via `bai-config.json` (include `"mongodb": "^7.1.1"`, and `"konva"` / `"react-konva"` when the Thunderstorm revision includes `@nu-art/konva-chart`).
- **Vite is the only app frontend** — do not add a webpack `app/frontend` tree.
- **UI requires a design language first** — human specifies `_docs/specs/design-language.md`; agent implements **`app-core/frontend`** (plus `app-core/shared`). No domain feature UI before that contract exists.
