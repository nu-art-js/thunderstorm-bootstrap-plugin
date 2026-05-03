---
name: bootstrap-thunderstorm-project
description: >-
  Scaffold a new Thunderstorm monorepo project from an empty directory.
  Use when creating a new standalone Thunderstorm project, initializing a BAI workspace,
  or when the user says "new thunderstorm project", "bootstrap project", or "create TS project from scratch".
---

# Bootstrap Thunderstorm Project

Workflow for creating a new Thunderstorm (BAI) monorepo project from scratch.
Based on Thunderstorm 0.500.x conventions.

## Step 1 — Collect project parameters

Use AskQuestion to gather the following. Provide sensible defaults where shown.

| Parameter | Example | Default |
|-----------|---------|---------|
| `projectName` | `my-cool-app` | *(required)* |
| `npmScope` | `@app` | `@app` |
| `gitRepoUrl` | `git@github.com:my-org/my-cool-app.git` | *(required)* |
| `nodeVersion` | `22.10.0` | `22.10.0` |
| `thunderstormRepo` | `git@github.com:nu-art-js/thunderstorm.git` | `git@github.com:nu-art-js/thunderstorm.git` |
| `thunderstormVersion` | `0.500.0` | latest tag on thunderstorm repo |
| `frontendType` | `vite` or `webpack` | `vite` |
| `license` | `Apache-2.0` | `Apache-2.0` |
| `author` | `My Org` | *(required)* |
| `firebaseProjectIds` | `{ local: "my-app-local", dev: "my-app-dev" }` | *(required — at least `local`)* |
| `initialPackages` | `["messaging/shared", "messaging/backend", "messaging/frontend"]` | `[]` |
| `port` | `8200` | `8000` |

Port derivation from `port` (N):

| Derived | Value | Used in |
|---------|-------|---------|
| `debugPort` | N | backend `unitConfig.debugPort` |
| `servingPort` | N + 1 | frontend `unitConfig.servingPort` |
| `basePort` | N + 2 | backend `unitConfig.basePort` |
| `configUrlPort` | N + 4 | frontend `envs.local.config.configUrl` |

## Step 2 — Initialize the repository

```bash
mkdir <projectName> && cd <projectName>
git init
git submodule add <thunderstormRepo> _thunderstorm
cd _thunderstorm && git checkout <thunderstormVersion-tag-or-branch> && cd ..
```

## Step 3 — Create root files

### `.nvmrc`

```
<nodeVersion>
```

### `build-and-install.sh`

```bash
#!/bin/bash

bash <(curl -fsSL https://github.com/nu-art/bash-tools/releases/latest/download/bundle.loader.sh) --sh-repo nu-art-js/build-and-install-script --sh-bundle bai "$@"
```

Make executable: `chmod +x build-and-install.sh`

### `version-app.json`

```json
{
  "version": "1.0.0"
}
```

### Root `__package.json`

See [reference.md](reference.md) for the full template. Key points:

- `"name"` — `"<npmScope>/<projectName>"` (e.g. `"@app/my-cool-app"`)
- `"version": "1.0.0"`, `"private": true`, `"type": "module"`
- `"engines.node"` — major version matching `.nvmrc` (e.g. `"22"`)
- `"unitConfig": { "type": "node-project" }`
- `"pnpm.overrides"` — include `"google-auth-library"` and `"@bcherny/json-schema-ref-parser"` overrides
- All thunderstorm/infra deps use `"?"` — BAI resolves from `bai-config.json`

### `bai-config.json`

See [reference.md](reference.md) for the full template. Key sections:

- `templateParams.packageJson` — all external dependency versions, plus:
  - `"THUNDERSTORM_VERSION": "<thunderstormVersion>"`
  - `"APP_VERSION": "1.0.0"`
- `files.firebase` — paths to `.config/.firebase-config/` files
- `files.typescript.tsConfig` — `main` and `test` point to `.config/tsconfig-main.json`
- `files.backend.proxy` — `null` (no proxy needed by default)
- `files.tests.firebase` — firebase test config paths (optional, add when tests are needed)

### `.config/tsconfig-main.json`

```json
{
  "compilerOptions": {
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "target": "ES2022",
    "jsx": "react-jsx",
    "allowJs": true,
    "lib": ["ES2022", "DOM"],
    "declaration": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "noUnusedLocals": true,
    "strict": true
  }
}
```

### `.config/eslint.config.cjs`

```javascript
const typescriptEslint = require("@typescript-eslint/eslint-plugin");
const tsParser = require("@typescript-eslint/parser");

module.exports = [
    {
        files:['**/*.ts','**/*.tsx'],
        plugins: {
            "@typescript-eslint": typescriptEslint,
        },
        languageOptions: {
            globals: {
                Atomics: "readonly",
                SharedArrayBuffer: "readonly",
            },
            parser: tsParser,
            ecmaVersion: 2018,
            sourceType: "module",
        },
        rules: {
            "no-mixed-spaces-and-tabs": "off",
            "no-unused-vars": "off",
            "no-useless-escape": "off",
            "no-inferrable-types": "off",
            "no-unnecessary-type-constraint": "off",
        },
    },
];
```

### `.config/.firebase-config/` files

Create four files with minimal defaults. See [reference.md](reference.md) for contents:

- `database.rules.json` — deny-all with `_config/frontend` readable
- `firestore.rules` — deny-all
- `firestore.indexes.json` — empty indexes
- `storage.rules` — deny-all

## Step 4 — Scaffold app packages

Apps live under the `app/` directory.

### Vite frontend: `app/frontend-vite/__package.json`

Key `unitConfig` fields for vite frontend:

```json
{
  "unitConfig": {
    "hasSelfHotReload": true,
    "servingPort": "N+1",
    "type": "vite-hosting",
    "label": "Frontend Vite",
    "envs": {
      "local": {
        "config": {
          "configUrl": "http://127.0.0.1:<N+4>/_config/frontend/manager.json?ns=<firebaseProjectIds.local>"
        },
        "projectId": "<firebaseProjectIds.local>"
      }
    }
  }
}
```

Scripts for vite:
```json
{
  "build": "vite build",
  "start": "vite"
}
```

Vite-specific devDependencies:
```json
{
  "@vitejs/plugin-react": "?",
  "vite": "?",
  "vite-plugin-node-polyfills": "?",
  "vite-plugin-svgr": "?",
  "sass": "?"
}
```

Core frontend dependencies (Thunderstorm 0.500.x):
```json
{
  "@nu-art/ts-common": "?",
  "@nu-art/thunder-core": "?",
  "@nu-art/thunder-routing": "?",
  "@nu-art/thunder-ui-modules": "?",
  "@nu-art/thunder-widgets": "?",
  "@nu-art/ts-styles": "?",
  "@nu-art/http-client": "?",
  "@nu-art/db-api-frontend": "?",
  "@nu-art/db-api-shared": "?",
  "@nu-art/firebase-frontend": "?",
  "@nu-art/sync-manager-frontend": "?",
  "@nu-art/editable-item": "?",
  "react": "?",
  "react-dom": "?",
  "react-router-dom": "?"
}
```

Create `app/frontend-vite/src/main/index.tsx` — minimal Thunder app entry. See [reference.md](reference.md).

### Backend: `app/backend/__package.json`

Key `unitConfig` fields:

```json
{
  "unitConfig": {
    "debugPort": "N",
    "basePort": "N+2",
    "functions": [
      {
        "name": "api",
        "trigger": "http",
        "resources": {
          "cpu": 1,
          "memory": "2Gi",
          "timeout": 540,
          "concurrency": 100,
          "minInstances": 0,
          "maxInstances": 3
        }
      }
    ],
    "type": "firebase-function",
    "label": "Backend",
    "sslKey": "../../.config/.ssl/localhost.key",
    "sslCert": "../../.config/.ssl/localhost.crt",
    "envs": {
      "local": { "projectId": "<firebaseProjectIds.local>" }
    }
  }
}
```

Core backend dependencies (Thunderstorm 0.500.x):
```json
{
  "@nu-art/ts-common": "?",
  "@nu-art/storm-core": "?",
  "@nu-art/http-server": "?",
  "@nu-art/api-types": "?",
  "@nu-art/db-api-backend": "?",
  "@nu-art/db-api-shared": "?",
  "@nu-art/firebase-backend": "?",
  "@nu-art/firebase-shared": "?",
  "@nu-art/sync-manager-backend": "?",
  "@nu-art/google-services-backend": "?",
  "@nu-art/permissions-backend": "?",
  "@nu-art/permissions-shared": "?",
  "@nu-art/user-account-backend": "?",
  "@nu-art/user-account-shared": "?",
  "@nu-art/backup-backend": "?",
  "@nu-art/action-processor-backend": "?",
  "firebase": "?",
  "firebase-admin": "?",
  "firebase-functions": "?",
  "express": "?"
}
```

Create `app/backend/src/main/index.ts` — minimal Storm app entry. See [reference.md](reference.md).

## Step 5 — Scaffold library packages

For each entry in `initialPackages` (e.g. `"messaging/shared"`):

1. Parse: `capability` = `messaging`, `domain` = `shared`
2. Create directory: `<capability>/<domain>/`
3. Create `__package.json`:

```json
{
  "name": "<npmScope>/<capability>-<domain>",
  "version": "{{APP_VERSION}}",
  "private": true,
  "description": "<capability> — <Domain>",
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

Add domain-specific thunderstorm deps:
- `shared` → `@nu-art/db-api-shared`
- `frontend` → `@nu-art/thunder-core`, `@nu-art/db-api-frontend`, `@nu-art/db-api-shared`, `@nu-art/http-client`, also dep on the sibling shared package
- `backend` → `@nu-art/db-api-backend`, `@nu-art/db-api-shared`, also dep on the sibling shared package

4. Create `src/main/index.ts` with a placeholder export
5. Create `src/test/` directory

## Step 6 — Additional environment configs

For each environment beyond `local` in `firebaseProjectIds`, add entries to the `envs` blocks in both app `__package.json` files.

## Step 7 — Create `.gitignore`

```
.fleet
.idea/*
!.idea/copyright/
!.idea/dictionaries/
!.idea/runConfigurations/
!.idea/scopes/

error
error_message.txt
.trash
.ts_env
**/node_modules

node_modules
pnpm-lock.yaml
pnpm-workspace.yaml
package.json

.firebaserc
.firebase
firebase.json
firebase-debug.log
package-lock.json

*.iml
**/.DS_Store
*.log
tsconfig.json
running-status.json
.cache
.config/.ssl
```

## Step 8 — Create `CLAUDE.md`

```markdown
# <projectName>

This project is built on the **Thunderstorm** framework.
Thunderstorm rules and non-negotiable principles are in [`_thunderstorm/CLAUDE.md`](_thunderstorm/CLAUDE.md).

## Build system

**Use BAI only.** Never run `pnpm` directly to build, test, or install.
BAI is a standalone CLI — it is not invoked via `pnpm run`.

- Commands and flags: [`_thunderstorm/.rules/operational/bai-cli.mdc`](_thunderstorm/.rules/operational/bai-cli.mdc)
- Project structure: [`_thunderstorm/.rules/operational/project-structure.mdc`](_thunderstorm/.rules/operational/project-structure.mdc)
```

## Step 9 — Create `_docs/` structure

```
_docs/
  specs/
  memories/
  troubleshooting/
```

## Step 10 — Initial setup

Run the initial-setup script created in Step 3:

```bash
bash build-and-install.sh init
```

Follow on the bai rules to make sure you understand how to use it!

For BAI CLI flag reference, read `_thunderstorm/.rules/operational/bai-cli.mdc`.

## Step 11 — Initial commit

Stage everything, commit with message: `"bootstrap: initial thunderstorm project scaffold"`.

## Key rules

- **Never edit generated files** — always edit `__package.json`, never `package.json` or `pnpm-workspace.yaml`
- **Use `?` for all dependency versions** — BAI resolves workspace deps and external versions from `bai-config.json`
- **`{{APP_VERSION}}`** — use this template variable in lib and app package versions; BAI substitutes from `bai-config.json`
- **ESM only** — all packages use `"type": "module"`
- **Package naming** — libs: `<npmScope>/<capability>-<domain>`, apps: `<npmScope>/<app-name>`
- **Source structure** — `src/main/` for source, `src/test/` for tests, always
- **Apps under `app/`** — `app/frontend-vite/` or `app/frontend/`, `app/backend/`
- **Use BAI only** — never run `pnpm` directly; always `bash ./build-and-install.sh [flags]`. See `_thunderstorm/.rules/operational/bai-cli.mdc` for the command reference.
