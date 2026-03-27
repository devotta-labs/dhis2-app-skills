# Bootstrapping a DHIS2 App

## Scaffolding

Create a new DHIS2 app with the official CLI:

```bash
pnpm create @dhis2/app@latest <app-name> --yes
```

The `--yes` flag accepts defaults: pnpm as package manager, JavaScript, and the basic template.

Other useful flags:
- `--typescript` / `--ts` — scaffold with TypeScript

Alternative package managers are supported but pnpm is recommended:
```bash
npm create @dhis2/app@latest <app-name> -- --yes
yarn create @dhis2/app <app-name> --yes
```

Note the extra `--` with npm — it's needed to pass flags through to the create script.

## d2.config.js

This file configures how the app integrates with DHIS2. Import the `D2Config` type from
`@dhis2/cli-app-scripts` for intellisense:

```javascript
/** @type {import('@dhis2/cli-app-scripts').D2Config} */
const config = {
    type: 'app',
    name: 'my-app',
    title: 'My Application',
    description: 'A DHIS2 application',
    minDHIS2Version: '2.40',

    entryPoints: {
        app: './src/App.tsx',
    },

    viteConfigExtensions: './vite.config.mts',
}

module.exports = config
```

Set `minDHIS2Version` to the oldest DHIS2 version you intend to support. This is required
for publishing to the App Hub.

## Vite Config Extensions

After scaffolding, create a `vite.config.mts` file in the project root. This is
merged into the App Platform's base Vite config automatically.

```typescript
import path from 'path'
import { defineConfig } from 'vite'

export default defineConfig({
    resolve: {
        alias: {
            '@': path.resolve(__dirname, 'src'),
        },
    },
})
```

This lets you import from `@/components/Foo` instead of `../../components/Foo`.

## Development Workflow

### Start the dev server

```bash
pnpm start
```

The app runs at `http://localhost:3000`. You'll see a login screen where you enter
the DHIS2 server URL and credentials.

### Connect to a remote instance

```bash
pnpm start --proxy https://play.im.dhis2.org/dev
```

The proxy runs on port 8080 and handles CORS automatically. At the login screen,
enter `http://localhost:8080` as the server URL with the credentials for that instance.

### Build for production

```bash
pnpm build
```

Output goes to `build/`. Upload the resulting zip to the DHIS2 App Hub or install
directly via the App Management app.

## App Platform v12 (Current)

The current App Platform uses Vite and React 18. Key things to know:

- **JSX files need `.jsx`/`.tsx` extensions** — Vite requires this. Plain `.js` files cannot
  contain JSX syntax.
- **Environment variables** use the `DHIS2_` prefix (not `REACT_APP_`). The old prefix still
  works but is deprecated.
- **Node 18 or 20+** is required.
- Use `window.variableName` instead of `global.variableName`.
