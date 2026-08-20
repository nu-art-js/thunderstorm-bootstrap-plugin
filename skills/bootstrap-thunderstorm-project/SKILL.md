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
| `firebaseProjectIds` | At least `local`; add `dev` / `staging` / `prod` if needed |
| `port` | Integer **N** — base for local ports (see [reference.md](reference.md)) |
| `keepFrontend` | Default true. Template frontend is **Vite only** (`app/frontend-vite`). If false (headless), delete that tree. |
| `initialPackages` | e.g. `["messaging/shared","messaging/backend","messaging/frontend"]` — new capability folders |
| `removeSampleCore` | If true: delete template `core/` and strip `@app/core-*` deps — **see Design language gate** if the project keeps a frontend |
| `specPath` | **Optional.** Absolute path or workspace-relative path to a product/architecture spec (markdown). User may paste or `@`-reference it. Read it **before** clone/customize to infer `projectName`, `initialPackages`, `keepFrontend`, phases, and integration notes. After bootstrap, optionally **copy** the spec into the new repo under `_docs/specs/` so the codebase carries its own contract. |
| `requiresUi` | **Implicitly true** when you keep `app/frontend-vite`. When true, a **design language spec** is mandatory before feature UI work; see Design language gate. |
| `designLanguageSpecPath` | **Optional.** Path to an existing design-language doc to copy into the new repo. If `requiresUi` and this is empty, **do not** implement screens or visual domain UI in bootstrap — create `_docs/specs/design-language.md` as a **human-blocked stub** (see reference.md). |

If `specPath` is set and `initialPackages` is empty, derive capability folders from the spec (sections, resource names, or explicit package list in the doc). If both conflict, **ask the user** which wins.

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

## Step 6 — Firebase project IDs and RTDB config URL

In **remaining** app `__package.json` files (`app/backend/`, `app/frontend-vite/` if kept):

- Replace placeholder `demo-project` / `real-project` with `firebaseProjectIds.local` (and other env keys if user provided them).
- In `envs.*.config.configUrl`, the `ns=` query segment must match the Firebase RTDB namespace for that project (pattern used in template: `<projectId>-default-rtdb`). Update `ns=` when `projectId` changes.

Add `envs` entries for `dev` / `staging` / `prod` mirroring the pattern in a mature project (see e.g. news-scraper) if the user supplied those IDs.

## Step 7 — Ports

Template defaults (before customization) use **N = 8000**:

| Role | Port |
|------|------|
| Backend `debugPort` | N |
| Frontend `servingPort` | N + 1 |
| Backend `basePort` | N + 2 |
| `configUrl` host port (`http://127.0.0.1:<port>/...`) | N + 4 |

Replace **8000, 8001, 8002, 8004** across the kept app packages with the user’s **N, N+1, N+2, N+4** if their `port` differs.

Keep **`app/e2e`**. It is a product consumer of `@nu-art/e2e-harness` (not another harness copy). Retarget its dedicated zone so it does not collide with human `bai -l` on **N+2**:

| Constant | Sample default | After port remap |
|----------|----------------|------------------|
| E2E backend (`SAMPLE_E2E_BACKEND_PORT`) | 8102 | a free port, typically **N+102** |
| E2E mongo (`SAMPLE_E2E_MONGO_PORT`) | 27039 | keep or pick an unused host port |
| E2E Firebase project id | `demo-project` | `firebaseProjectIds.local` |

Also search for `localhost:8008` (storage emulator proxy in backend) only if you intentionally change storage proxy wiring — default leave as-is unless documented otherwise.

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

## Step 11 — Dev workspace launcher config (recommended for local dev)

If the project is developed locally with multiple long-running processes (watch,
backend, one or more frontends), scaffold a dev workspace so the machine-wide
`launch-workspace` skill (repo: `agent-skill--launch-workspace`) can open them as
one Ghostty split grid. No per-project script or schema is copied — only a
manifest plus a config declaration.

1. **Create the manifest** at `.cursor/dev-workspaces/workspaces.json`:
   ```json
   {
     "$schema": "https://raw.githubusercontent.com/nu-art-js/agent-skill--launch-workspace/main/skills/launch-workspace/workspaces.schema.json",
     "workspaces": {
       "<projectName>-dev": {
         "title": "<projectName> Dev",
         "root": ".",
         "panes": [
           { "label": "watch",    "command": "bai -w -wbt" },
           { "label": "backend",  "command": "sleep 1 && bai -nb -up=@<npmScope>/<app>-backend -l" },
           { "label": "frontend", "command": "sleep 2 && bai -nb -up=@<npmScope>/frontend-vite -lf" }
         ]
       }
     }
   }
   ```
   Stagger non-watch panes with `sleep N &&` so services don't race on ports/build
   at the same instant. Pick the pane set to match the kept apps.

2. **Declare it** in `.cursor/project-config.yaml` (the SSOT for project junctions;
   the agent reads this to locate the manifest — it is not hardcoded convention):
   ```yaml
   dev-workspaces:
     manifest: .cursor/dev-workspaces/workspaces.json
     skill: launch-workspace
   ```

3. **Gitignore the logs** — the launcher writes per-pane logs under
   `.cursor/scratch/`. Ensure `.cursor/scratch/` (and `*.log`) are gitignored.

The `launch-workspace` skill must be installed machine-wide (`~/.cursor/skills/`)
for this to run; see that skill for layouts, `--status`, logs, and reconcile.

## Step 12 — Initial setup

From project root, run a **full** workspace init **through the repo script**:

```bash
bash build-and-install.sh init
```

This is the **only** correct bootstrap command for a fresh clone — it creates a temporary `package.json`, installs tsx via pnpm, then runs the full BAI pipeline. **Do not** run `initial-setup.sh` or pass raw flags like `-fs -th -p -cox` — those assume tsx is already installed and will fail on a fresh clone.

The sample’s `build-and-install.sh` injects `--ts-version` from `version-thunderstorm.json` (0.500.x). The **upstream** BAI wrapper still defaults to `~0.401.0` and does **not** read that file.

**Never** `bai -i -up=<subset>` (or `bash build-and-install.sh -i -up=…`). A subset install rewrites a broken `pnpm-workspace.yaml` and drops the rest of the monorepo. Use full-workspace `init` or `bai -i -nb` only.

Never use raw `pnpm install` / `pnpm run build` as the primary workflow — BAI owns the lifecycle.

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
- **Pin Thunderstorm 0.500.x** — SSOT is `version-thunderstorm.json` (and `bai-config.json` `THUNDERSTORM_VERSION`). The sample `build-and-install.sh` injects `--ts-version` from that file. **Verify** `node_modules/@nu-art/build-and-install` is 0.500.x after init. If it is 0.401.x, use the `--ts-version=<pin>` / `TS_VERSION=<pin>` hack and re-init. Upstream BAI still defaults to `~0.401.0`.
- **Docker must be running** before `bai -l` or product e2e. **`GCP_PROJECT_ID`** must be a real GCP project before password-auth / JWT.
- **Never edit BAI-generated** `package.json` / `pnpm-workspace.yaml` — only `__package.json` templates and source; run BAI to regenerate.
- **Never subset-install** — `bai -i -up=<subset>` rewrites a broken workspace file. Full `init` / `bai -i -nb` only.
- **Keep `@app/e2e`** — retarget ports and Firebase project id; do not add another `@app/e2e-harness` copy (use `@nu-art/e2e-harness`).
- **`?` and `{{APP_VERSION}}`** — keep template conventions; versions resolve via `bai-config.json` (include `"mongodb": "^7.1.1"`, and `"konva"` / `"react-konva"` when the Thunderstorm revision includes `@nu-art/konva-chart`).
- **Vite is the only app frontend** — do not add a webpack `app/frontend` tree.
- **UI requires a design language first** — human specifies `_docs/specs/design-language.md`; agent implements **`app-core/frontend`** (plus `app-core/shared`). No domain feature UI before that contract exists.
