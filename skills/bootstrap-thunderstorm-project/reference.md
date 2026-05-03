# Reference — Bootstrap Thunderstorm Project (post-clone)

Assumes the repo was cloned from `thunderstorm-sample` (`nu-art-js/thunderstorm-sample`).

## Port map (template baseline N = 8000)

| Field | Formula | Default |
|-------|---------|---------|
| Backend `unitConfig.debugPort` | N | 8000 |
| Backend `unitConfig.basePort` | N + 2 | 8002 |
| Frontend `unitConfig.servingPort` | N + 1 | 8001 |
| Webpack `webpackConfig.devServerPort` | N + 1 | 8001 |
| `envs.local.config.configUrl` | `http://127.0.0.1:<N+4>/...` | port **8004** |

Grep for `8000`, `8001`, `8002`, `8004` in `app/**/__package.json` after choosing the kept frontend.

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
| `frontend` | `@app/<capability>-shared`, `@nu-art/thunder-core`, `@nu-art/db-api-frontend`, `@nu-art/db-api-shared`, `@nu-art/http-client` |
| `backend` | `@app/<capability>-shared`, `@nu-art/db-api-backend`, `@nu-art/db-api-shared` |

## Minimal `src/main/index.ts` (library)

```typescript
export {};
```

Or a single named export placeholder.

## RTDB `configUrl` / `projectId`

Local pattern from template:

- `"projectId": "<your-local-firebase-project-id>"`
- `"configUrl": "http://127.0.0.1:<N+4>/_config/frontend/manager.json?ns=<your-local-firebase-project-id>-default-rtdb"`

## Removing `core/`

Dependencies to remove from `app/backend/__package.json`:

- `@app/core-shared`
- `@app/core-backend`

Search frontends for `@app/core-` and remove. Then delete `core/shared`, `core/frontend`, `core/backend` directories.
