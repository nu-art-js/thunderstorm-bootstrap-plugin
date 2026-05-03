# Reference — Bootstrap Thunderstorm Project

Complete templates for project bootstrapping. Thunderstorm 0.500.x standard.

## Root `__package.json` template

```json
{
  "name": "<npmScope>/<projectName>",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "scripts": {},
  "devDependencies": {
    "tsx": "^4.20.5",
    "@types/chai": "^4.3.4",
    "@types/chai-as-promised": "^7.1.5",
    "@types/mocha": "^10.0.1",
    "@types/node": "?",
    "@types/estree": "^1.0.0",
    "@typescript-eslint/eslint-plugin": "?",
    "@typescript-eslint/parser": "?",
    "@nu-art/build-and-install": "?",
    "@nu-art/cli-params": "?",
    "@nu-art/logger": "?",
    "@nu-art/commando": "?",
    "@nu-art/ts-common": "?",
    "@types/express": "?",
    "firebase": "?",
    "firebase-admin": "?",
    "firebase-functions": "?",
    "google-auth-library": "?",
    "chai": "^4.3.7",
    "chai-as-promised": "^7.1.1",
    "eslint": "?",
    "mocha": "^10.2.0",
    "ts-mocha": "^10.0.0",
    "typedoc": "^0.23.28",
    "typescript": "?",
    "use-strict": "^1.0.1",
    "tsc-watch": "^6.0.0",
    "undici-types": "^7.8.0",
    "firebase-tools": "?",
    "ts-node": "latest",
    "vite": "?",
    "jose": "?",
    "moment": "?",
    "vite-plugin-node-polyfills": "?"
  },
  "pnpm": {
    "overrides": {
      "google-auth-library": "10.3.0",
      "@bcherny/json-schema-ref-parser": "npm:@apidevtools/json-schema-ref-parser@^14.2.0"
    }
  },
  "engines": {
    "node": "<nodeVersion-major>"
  },
  "unitConfig": {
    "type": "node-project"
  }
}
```

## `bai-config.json` template

```json
{
  "templateParams": {
    "packageJson": {
      "moment": "^2.29.4",
      "firebase": "^11.9.0",
      "firebase-admin": "13.4.0",
      "firebase-functions": "6.3.2",
      "firebase-tools": "latest",
      "react": "^18.0.0",
      "@types/react": "^18.0.0",
      "react-dom": "^18.0.0",
      "@types/react-dom": "^18.0.0",
      "react-router-dom": "^6.9.0",
      "@types/react-router-dom": "^5.3.3",
      "@types/react-router": "^5.1.20",
      "@types/chai": "^4.3.4",
      "chai": "^4.3.4",
      "@types/mocha": "^10.0.1",
      "express": "^4.18.2",
      "@types/express": "^4.17.17",
      "@types/node": "^22.0.0",
      "typescript": "^5.0.0",
      "eslint": "9.18.0",
      "@typescript-eslint/parser": "8.19.1",
      "@typescript-eslint/eslint-plugin": "8.19.1",
      "sass": "^1.89.0",
      "jose": "^5.0.0",
      "openai": "^6.7.0",
      "vite": "^6.0.0",
      "@vitejs/plugin-react": "^4.5.0",
      "vite-plugin-node-polyfills": "^0.25.0",
      "vite-plugin-svgr": "^4.3.0",
      "@google-cloud/common": "^5.0.2",
      "@google-cloud/storage": "^7.15.0",
      "@google-cloud/firestore": "^7.11.0",
      "@google-cloud/pubsub": "^4.0.0",
      "@google-cloud/secret-manager": "^6.1.0",
      "google-auth-library": "^10.0.0",
      "googleapis": "^152.0.0",
      "axios": "^1.13.1",
      "THUNDERSTORM_VERSION": "<thunderstormVersion>",
      "APP_VERSION": "1.0.0"
    }
  },
  "files": {
    "firebase": {
      "databaseRules": "./.config/.firebase-config/database.rules.json",
      "storageRules": "./.config/.firebase-config/storage.rules",
      "firestoreIndexesRules": "./.config/.firebase-config/firestore.indexes.json",
      "firestoreRules": "./.config/.firebase-config/firestore.rules"
    },
    "typescript": {
      "tsConfig": {
        "main": "./.config/tsconfig-main.json",
        "test": "./.config/tsconfig-main.json"
      }
    },
    "backend": {
      "proxy": null
    }
  }
}
```

## Vite frontend app `__package.json` template

`app/frontend-vite/__package.json`:

```json
{
  "name": "<npmScope>/frontend-vite",
  "version": "{{APP_VERSION}}",
  "private": true,
  "description": "<projectName> Frontend",
  "license": "<license>",
  "author": "<author>",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "vite build",
    "start": "vite"
  },
  "dependencies": {
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
    "@nu-art/permissions-frontend": "?",
    "@nu-art/permissions-shared": "?",
    "@nu-art/user-account-frontend": "?",
    "@nu-art/user-account-shared": "?",
    "@nu-art/ts-focused-object-frontend": "?",
    "@nu-art/ts-focused-object-shared": "?",
    "@nu-art/thunder-mouse-interactivity-frontend": "?",
    "react": "?",
    "react-dom": "?",
    "react-router-dom": "?"
  },
  "devDependencies": {
    "@types/react": "?",
    "@types/react-dom": "?",
    "@types/react-router-dom": "?",
    "@vitejs/plugin-react": "?",
    "sass": "?",
    "vite": "?",
    "vite-plugin-node-polyfills": "?",
    "vite-plugin-svgr": "?"
  },
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
  },
  "type": "module",
  "publishConfig": {
    "directory": "dist"
  },
  "exports": {
    ".": {
      "types": "./index.d.ts",
      "import": "./index.js"
    }
  }
}
```

## Backend app `__package.json` template

`app/backend/__package.json`:

```json
{
  "name": "<npmScope>/backend",
  "version": "{{APP_VERSION}}",
  "private": true,
  "description": "<projectName> Backend",
  "license": "<license>",
  "author": "<author>",
  "scripts": {
    "build": "tsc",
    "serve": "bash launch-server.sh",
    "launch": "bash launch-server.sh",
    "debug": "bash launch-server.sh --debug"
  },
  "dependencies": {
    "@nu-art/ts-common": "?",
    "@nu-art/storm-core": "?",
    "@nu-art/http-server": "?",
    "@nu-art/api-types": "?",
    "@nu-art/db-api-backend": "?",
    "@nu-art/db-api-shared": "?",
    "@nu-art/firebase-backend": "?",
    "@nu-art/firebase-shared": "?",
    "@nu-art/google-services-backend": "?",
    "@nu-art/sync-manager-backend": "?",
    "@nu-art/permissions-backend": "?",
    "@nu-art/permissions-shared": "?",
    "@nu-art/user-account-backend": "?",
    "@nu-art/user-account-shared": "?",
    "@nu-art/backup-backend": "?",
    "@nu-art/action-processor-backend": "?",
    "firebase": "?",
    "firebase-admin": "?",
    "firebase-functions": "?",
    "express": "?",
    "moment": "?"
  },
  "devDependencies": {
    "@types/node-fetch": "^2.5.7"
  },
  "engines": {
    "node": "<nodeVersion-major>"
  },
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
      "local": {
        "projectId": "<firebaseProjectIds.local>"
      }
    }
  },
  "type": "module",
  "exports": {
    ".": { "types": "./index.d.ts", "import": "./index.js" },
    "./*": { "types": "./*.d.ts", "import": "./*.js" }
  },
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts"
}
```

## Minimal frontend entry point

`app/frontend-vite/src/main/index.tsx`:

```typescript
import {Thunder} from '@nu-art/thunder-core';
import {ThunderstormDefaultApp} from '@nu-art/thunder-widgets';
import type {Module} from '@nu-art/ts-common';
import {ModuleFE_FirebaseListener} from '@nu-art/firebase-frontend';
import {ModuleFE_SyncManager} from '@nu-art/sync-manager-frontend';
import {ModuleFE_MouseInteractivity, TS_ToolTip, TS_PopUp} from '@nu-art/thunder-mouse-interactivity-frontend';
import {ModulePackFE_Accounts} from '@nu-art/user-account-frontend';
import {ModulePackFE_Permissions} from '@nu-art/permissions-frontend';
import {ModulePackFE_FocusedObject} from '@nu-art/ts-focused-object-frontend';
import {deriveConfig} from './app-config.js';
import {Route_Page_Main} from './ui/pages/Page_Main/route.js';
import './App.scss';

const modulePackFE_Thunderstorm: Module[] = [
	ModuleFE_FirebaseListener,
	ModuleFE_SyncManager,
	ModuleFE_MouseInteractivity,
];

new Thunder(deriveConfig())
	.addModulePack(modulePackFE_Thunderstorm)
	.addModulePack(ModulePackFE_Accounts)
	.addModulePack(ModulePackFE_Permissions)
	.addModulePack(ModulePackFE_FocusedObject)
	.setMainApp(ThunderstormDefaultApp, {rootRoute: Route_Page_Main, additionalOverlays: [TS_ToolTip, TS_PopUp]})
	.build();
```

## Minimal backend entry point

`app/backend/src/main/index.ts`:

```typescript
import {HttpServer} from '@nu-art/http-server';
import {Storm} from '@nu-art/storm-core';
import {DebugFlag, LogLevel, Module} from '@nu-art/ts-common';
import {ModuleBE_Permissions, ModuleBE_PermissionsAssert, ModulePackBE_Permissions} from '@nu-art/permissions-backend';
import {ModuleBE_ExpressFunction_Class} from '@nu-art/firebase-backend';
import {ModuleBE_Auth} from '@nu-art/google-services-backend';
import {ApiDef_UserAccount} from '@nu-art/user-account-shared';
import {ModuleBE_AccountDB, ModuleBE_SessionDB, ModulePackBE_Accounts_WOSAML} from '@nu-art/user-account-backend';
import {ModuleBE_SyncManager} from '@nu-art/sync-manager-backend';
import {ModuleBE_BackupScheduler} from '@nu-art/backup-backend';
import {ModulePackBE_FocusedObject} from '@nu-art/ts-focused-object-backend';
import {runProjectSetup} from '@nu-art/action-processor-backend';
import {ModuleBE_BaseDB} from '@nu-art/db-api-backend';
import type {ApiDef} from '@nu-art/api-types';
import {Environment} from './config.js';

DebugFlag.DefaultLogLevel = LogLevel.Verbose;
ModuleBE_BaseDB.setDefaultBackend('mongo');
ModuleBE_BackupScheduler.setDefaultConfig({memory: '1GB'});

const ModuleBE_ApiFunction = new ModuleBE_ExpressFunction_Class('api', () => HttpServer.getDefault().getExpress());
ModuleBE_ApiFunction.setDefaultConfig({
	options: {
		cors: false,
		memory: '1GiB',
		timeoutSeconds: 540,
		maxInstances: 10,
		concurrency: 100,
	}
});

const modules: Module[] = [
	ModuleBE_Auth,
	ModuleBE_SyncManager,
];

const openApis = [
	ApiDef_UserAccount.registerAccount,
	ApiDef_UserAccount.login,
	ApiDef_UserAccount.getPasswordAssertionConfig,
];

HttpServer.getDefault().addApiMiddleware(
	(apiDef: ApiDef<any>) => !openApis.includes(apiDef),
	ModuleBE_SessionDB.Middleware,
	ModuleBE_AccountDB.Middleware,
	ModuleBE_PermissionsAssert.LoadPermissionsMiddleware
);

const stormConfig = {
	envKey: Environment.envKey,
	pathToDefaultConfig: `/_config/default`,
	pathToEnvOverrideConfig: `/_config/app`,
};

let _exposedFunctions = new Storm(stormConfig)
	.addModulePack([ModuleBE_ApiFunction])
	.addModulePack(ModulePackBE_Accounts_WOSAML)
	.addModulePack(ModulePackBE_FocusedObject)
	.addModulePack(ModulePackBE_Permissions)
	.addModulePack(modules)
	.addPostBuildAction(runProjectSetup)
	.build();

export const api = _exposedFunctions['api'];
```

## Minimal backend config

`app/backend/src/main/config.ts`:

```typescript
export const Environment = {
	envKey: process.env.GCLOUD_PROJECT ?? 'local',
};
```

## Firebase config file defaults

### `database.rules.json`

```json
{
  "rules": {
    ".read": false,
    ".write": false,
    "_config": {
      "frontend": {
        ".read": true,
        ".write": false
      }
    },
    "state": {
      "ModuleBE_SyncManager": {
        "syncData": {
          ".read": true,
          ".write": false
        }
      },
      "ModuleBE_FocusedObject": {
        "focusedData": {
          ".read": true,
          ".write": false
        }
      }
    }
  }
}
```

### `firestore.rules`

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

### `firestore.indexes.json`

```json
{
  "indexes": [],
  "fieldOverrides": []
}
```

### `storage.rules`

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if false;
    }
  }
}
```

## `.gitmodules`

```ini
[submodule "_thunderstorm"]
	path = _thunderstorm
	url = <thunderstormRepo>
```

## Unit type reference

| Unit type | `unitConfig.type` | Purpose | Key fields |
|-----------|-------------------|---------|------------|
| `node-project` | `"node-project"` | Root package — one per repo | `pnpm.overrides`, `engines.node` |
| `typescript-lib` | `"typescript-lib"` | Shared/frontend/backend library | `publishConfig`, `exports` |
| `vite-hosting` | `"vite-hosting"` | Vite frontend app | `servingPort`, `envs`, `hasSelfHotReload`, `label` |
| `firebase-hosting` | `"firebase-hosting"` | Webpack frontend app | `servingPort`, `envs`, `hasSelfHotReload`, `customTSConfig` |
| `firebase-function` | `"firebase-function"` | Backend app | `basePort`, `debugPort`, `functions[]`, `sslKey`, `sslCert`, `envs`, `label` |

## Thunderstorm 0.500.x package mapping

Frontend:
| Package | Purpose |
|---------|---------|
| `@nu-art/thunder-core` | Core frontend framework (Thunder class, module system) |
| `@nu-art/thunder-routing` | Route definitions and navigation |
| `@nu-art/thunder-ui-modules` | UI module infrastructure |
| `@nu-art/thunder-widgets` | Reusable UI components (ThunderstormDefaultApp, etc.) |
| `@nu-art/thunder-mouse-interactivity-frontend` | Tooltips, popups, mouse interactions |
| `@nu-art/ts-styles` | Style utilities |
| `@nu-art/http-client` | Frontend HTTP client |
| `@nu-art/db-api-frontend` | Frontend DB API cache/dispatchers |
| `@nu-art/firebase-frontend` | Firebase frontend listener |
| `@nu-art/sync-manager-frontend` | Data sync manager |
| `@nu-art/editable-item` | Editable item pattern |

Backend:
| Package | Purpose |
|---------|---------|
| `@nu-art/storm-core` | Core backend framework (Storm class, module system) |
| `@nu-art/http-server` | Express HTTP server |
| `@nu-art/api-types` | API definition types |
| `@nu-art/db-api-backend` | Backend DB API (Firestore/Mongo) |
| `@nu-art/firebase-backend` | Firebase admin, functions, storage |
| `@nu-art/google-services-backend` | GCP auth, service accounts |
| `@nu-art/sync-manager-backend` | Backend sync manager |
| `@nu-art/backup-backend` | Backup scheduler |
| `@nu-art/action-processor-backend` | Post-build actions, project setup |

Shared:
| Package | Purpose |
|---------|---------|
| `@nu-art/ts-common` | Core utilities, types, base module |
| `@nu-art/db-api-shared` | DB entity types, API definitions |
| `@nu-art/permissions-shared` | Permission types |
| `@nu-art/user-account-shared` | Account API definitions |
| `@nu-art/firebase-shared` | Firebase shared types |
