---
name: bootstrap-thunderstorm-project
description: >-
  Scaffold a new Thunderstorm monorepo by cloning the official sample boilerplate,
  then renaming, re-pointing git, adjusting ports and Firebase project IDs, and
  optionally adding capability packages. Use for "new thunderstorm project",
  "bootstrap BAI monorepo", or "clone thunderstorm template".
---

# Bootstrap Thunderstorm Project (clone-based)

Do **not** hand-author the full monorepo from scratch. Start from the maintained boilerplate, then customize.

**Source of truth:** `git@github.com:nu-art-js/thunderstorm-sample.git` (Thunderstorm 0.500.x, Vite + Webpack frontends, sample `core/` library, BAI, `_thunderstorm` submodule).

After clone, read `_thunderstorm/.rules/operational/bai-cli.mdc` and `_thunderstorm/.rules/operational/project-structure.mdc` before running BAI.

## Step 1 — Collect parameters

| Parameter | Notes |
|-----------|--------|
| `projectName` | Directory name and human label |
| `npmScope` | e.g. `@app` — must match how you name workspace packages |
| `gitRepoUrl` | New repo URL (`git@github.com:org/repo.git`) |
| `templateRepoUrl` | Default: `git@github.com:nu-art-js/thunderstorm-sample.git` |
| `thunderstormCheckout` | Optional: tag or branch for `_thunderstorm` after clone (default: keep template’s submodule pointer) |
| `author`, `license` | For `__package.json` / package metadata where applicable |
| `firebaseProjectIds` | At least `local`; add `dev` / `staging` / `prod` if needed |
| `port` | Integer **N** — base for local ports (see [reference.md](reference.md)) |
| `frontendType` | `vite` or `webpack` — **delete the other** `app/frontend*` tree after clone |
| `initialPackages` | e.g. `["messaging/shared","messaging/backend","messaging/frontend"]` — new capability folders |
| `removeSampleCore` | If true: delete `core/` and strip `@app/core-*` deps and imports (search `@app/core-`) |

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

## Step 9 — Optional: remove sample `core/` library

If `removeSampleCore` is true:

1. Delete `core/` (all three domains).
2. Remove `@app/core-shared` and `@app/core-backend` from `app/backend/__package.json` dependencies.
3. Remove any `@app/core-frontend` / `@app/core-shared` from the kept frontend `__package.json`.
4. `grep -R "@app/core-" app/` and fix imports / module packs.

## Step 10 — Add `initialPackages`

For each path like `messaging/shared`:

1. Create `messaging/shared/`, `messaging/frontend/`, etc. as needed.
2. Add `__package.json` per [reference.md](reference.md) (typescript-lib pattern, `{{APP_VERSION}}`, exports).
3. Add minimal `src/main/index.ts` (and `src/test/` if tests expected).
4. Wire the new packages into the **kept** app `__package.json` dependencies so BAI discovers them in the graph.

## Step 11 — Initial setup

From project root, run whatever first-time script the template ships (e.g. `bash initial-setup.sh` if present). Otherwise follow `_thunderstorm/.rules/operational/bai-cli.mdc` for install/build.

Never use raw `pnpm install` / `pnpm run build` as the primary workflow — BAI owns the lifecycle.

## Step 12 — Commit

Commit with a clear message (e.g. `bootstrap: <projectName> from thunderstorm-sample`).

## Key rules

- **Clone first** — boilerplate lives in `thunderstorm-sample`; the skill customizes, it does not recreate the tree file-by-file.
- **Never edit BAI-generated** `package.json` / `pnpm-workspace.yaml` — only `__package.json` templates and source; run BAI to regenerate.
- **`?` and `{{APP_VERSION}}`** — keep template conventions; versions resolve via `bai-config.json`.
- **One frontend app** in active development — remove the unused `app/frontend` or `app/frontend-vite` tree to avoid duplicate workspace units.
