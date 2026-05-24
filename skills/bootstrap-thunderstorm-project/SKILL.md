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

**Source of truth:** `git@github.com:nu-art-js/thunderstorm-sample.git` (Thunderstorm 0.500.x, Vite + Webpack frontends, sample `core/` library, BAI, `_thunderstorm` submodule).

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
| `frontendType` | `vite` or `webpack` — **delete the other** `app/frontend*` tree after clone |
| `initialPackages` | e.g. `["messaging/shared","messaging/backend","messaging/frontend"]` — new capability folders |
| `removeSampleCore` | If true: delete template `core/` and strip `@app/core-*` deps — **see Design language gate** if the project keeps a frontend |
| `specPath` | **Optional.** Absolute path or workspace-relative path to a product/architecture spec (markdown). User may paste or `@`-reference it. Read it **before** clone/customize to infer `projectName`, `initialPackages`, `frontendType`, phases, and integration notes. After bootstrap, optionally **copy** the spec into the new repo under `_docs/specs/` so the codebase carries its own contract. |
| `requiresUi` | **Implicitly true** when you keep `app/frontend-vite` or `app/frontend` (any real UI). When true, a **design language spec** is mandatory before feature UI work; see Design language gate. |
| `designLanguageSpecPath` | **Optional.** Path to an existing design-language doc to copy into the new repo. If `requiresUi` and this is empty, **do not** implement screens or visual domain UI in bootstrap — create `_docs/specs/design-language.md` as a **human-blocked stub** (see reference.md). |

If `specPath` is set and `initialPackages` is empty, derive capability folders from the spec (sections, resource names, or explicit package list in the doc). If both conflict, **ask the user** which wins.

## Design language gate (projects with UI)

Applies whenever the monorepo keeps a Thunderstorm frontend app (`vite-hosting` or `firebase-hosting`).

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

## Step 2 — Clone

```bash
git clone <templateRepoUrl> <projectName>
cd <projectName>
```

Optional: `cd _thunderstorm && git fetch && git checkout <thunderstormCheckout> && cd ..` then commit submodule pointer if you changed it.

## Step 3 — Point git at the new remote

```bash
git remote set-url origin <gitRepoUrl>
```

Keep history from the sample, or `rm -rf .git && git init` if you want a clean history (then add remote and initial commit).

## Step 4 — Choose frontend; delete the other

| `frontendType` | Keep | Remove |
|----------------|------|--------|
| `vite` | `app/frontend-vite/` | `app/frontend/` (webpack) |
| `webpack` | `app/frontend/` | `app/frontend-vite/` |

Delete the entire unused tree (including `__package.json`). Do not leave orphan packages in workspace.

## Step 5 — Rename workspace root package

In root `__package.json`:

- Set `"name"` from `@app/sample-project` to `<npmScope>/<projectName>` (or `<npmScope>/<slug>` if npm name must differ from folder).

Regenerate workspace metadata with BAI after edits (see Step 9).

## Step 6 — Firebase project IDs and RTDB config URL

In **remaining** app `__package.json` files (`app/backend/`, kept frontend):

- Replace placeholder `demo-project` / `real-project` with `firebaseProjectIds.local` (and other env keys if user provided them).
- In `envs.*.config.configUrl`, the `ns=` query segment must match the Firebase RTDB namespace for that project (pattern used in template: `<projectId>-default-rtdb`). Update `ns=` when `projectId` changes.

Add `envs` entries for `dev` / `staging` / `prod` mirroring the pattern in a mature project (see e.g. news-scraper) if the user supplied those IDs.

## Step 7 — Ports

Template defaults (before customization) use **N = 8000**:

| Role | Port |
|------|------|
| Backend `debugPort` | N |
| Frontend `servingPort` (and webpack `devServerPort` if applicable) | N + 1 |
| Backend `basePort` | N + 2 |
| `configUrl` host port (`http://127.0.0.1:<port>/...`) | N + 4 |

Replace **8000, 8001, 8002, 8004** across the kept app packages with the user’s **N, N+1, N+2, N+4** if their `port` differs.

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

## Step 11 — Initial setup

From project root, run `bash build-and-install.sh init`. This is the **only** correct bootstrap command for a fresh clone — it creates a temporary `package.json`, installs tsx via pnpm, then runs the full BAI pipeline. **Do not** run `initial-setup.sh` or pass raw flags like `-fs -th -p -cox` — those assume tsx is already installed and will fail on a fresh clone.

Never use raw `pnpm install` / `pnpm run build` as the primary workflow — BAI owns the lifecycle.

## Step 12 — Commit

Commit with a clear message (e.g. `bootstrap: <projectName> from thunderstorm-sample`).

## Key rules

- **Clone first** — boilerplate lives in `thunderstorm-sample`; the skill customizes, it does not recreate the tree file-by-file.
- **Never edit BAI-generated** `package.json` / `pnpm-workspace.yaml` — only `__package.json` templates and source; run BAI to regenerate.
- **`?` and `{{APP_VERSION}}`** — keep template conventions; versions resolve via `bai-config.json`.
- **One frontend app** in active development — remove the unused `app/frontend` or `app/frontend-vite` tree to avoid duplicate workspace units.
- **UI requires a design language first** — human specifies `_docs/specs/design-language.md`; agent implements **`app-core/frontend`** (plus `app-core/shared`). No domain feature UI before that contract exists.
