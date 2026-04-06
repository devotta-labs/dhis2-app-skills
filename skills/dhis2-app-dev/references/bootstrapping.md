# Bootstrapping a DHIS2 App

This is the step-by-step recipe for setting up a new DHIS2 app with the opinionated
tech stack. Follow every step in order — don't skip or substitute.

**Tech stack:** TypeScript, pnpm, React Router DOM, TanStack Query v4, TanStack Table,
`@` path alias, Vite.

---

## Step 1: Check for a local DHIS2 instance

Before scaffolding, check whether the user has DHIS2 running locally on the default port:

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/api/system/info
```

- **200** — local instance is running. Use `http://localhost:8080` as the server URL during development.
- **Any other response or connection error** — no instance detected on port 8080. This doesn't
  mean the user has no local instance — it may be on a different port, or they may use a remote
  instance. Ask the user which DHIS2 instance they want to develop against.

## Step 2: Scaffold

```bash
pnpm create @dhis2/app@latest <app-name> --typescript --yes
```

Always use `--typescript`. The `--yes` flag accepts defaults (pnpm, basic template).

```bash
cd <app-name>
```

## Step 3: Install the stack

```bash
pnpm add @tanstack/react-query@4 @tanstack/react-table react-router-dom
```

TanStack Query must be version 4 — do not install v5.

## Step 4: Configure `d2.config.js`

Replace the scaffolded config with:

```javascript
/** @type {import('@dhis2/cli-app-scripts').D2Config} */
const config = {
    type: 'app',
    name: '<app-name>',
    title: '<App Title>',
    description: '<short description>',
    minDHIS2Version: '2.40',

    entryPoints: {
        app: './src/App.tsx',
    },

    viteConfigExtensions: './vite.config.mts',
}

module.exports = config
```

Set `minDHIS2Version` to the oldest DHIS2 version the app needs to support.

## Step 5: Create `vite.config.mts`

Create this file in the project root:

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

This enables `@/components/Foo` imports instead of relative paths.

## Step 6: Add the path alias to `tsconfig.json`

Add the `paths` mapping so TypeScript resolves the `@` alias:

```json
{
    "compilerOptions": {
        "jsx": "react-jsx", // Overwrite the default jsx setting from the scaffolder
        "paths": {
            "@/*": ["./src/*"]
        }
    }
}
```

Merge this into the existing `tsconfig.json` — don't overwrite the other compiler options
that the scaffolder set up.

## Step 7: Create `src/utils/SyncUrlWithGlobalShell.tsx`

```tsx
import { useEffect } from 'react'
import { Outlet, useLocation } from 'react-router-dom'

/*
 * When the app runs in the DHIS2 Global Shell, react-router@6+ no longer
 * fires "popstate" events on pushState/replaceState. The Global Shell
 * listens for "popstate" to keep the browser URL in sync, so we dispatch
 * it manually on every route change.
 *
 * Background on the react-router change:
 * https://github.com/remix-run/react-router/blob/44472360ec9ea045008f453280bb749cb58e90ea/decisions/0005-remixing-react-router.md#inline-the-history-library-into-the-router
 */

export const SyncUrlWithGlobalShell = () => {
    const location = useLocation()

    useEffect(() => {
        dispatchEvent(new PopStateEvent('popstate'))
    }, [location.key])

    return <Outlet />
}
```

This is a layout route component — it wraps all routes so the Global Shell URL stays
in sync. Without it, the browser URL won't update when navigating. Every route should
be a child of this layout.

## Step 8: Set up `src/App.tsx`

Replace the contents of `src/App.tsx` with:

```tsx
import React from 'react'
import { createHashRouter, RouterProvider } from 'react-router-dom'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { CssReset, CssVariables } from '@dhis2/ui'
import { SyncUrlWithGlobalShell } from '@/utils/SyncUrlWithGlobalShell'

const queryClient = new QueryClient()

const router = createHashRouter([
    {
        element: <SyncUrlWithGlobalShell />,
        children: [
            {
                path: '/',
                element: <div>Home</div>,
            },
        ],
    },
])

const App = () => (
    <QueryClientProvider client={queryClient}>
        <CssReset />
        <CssVariables theme spacers colors elevations />
        <RouterProvider router={router} />
    </QueryClientProvider>
)

export default App
```

DHIS2 apps run inside an iframe in the DHIS2 shell, so use `createHashRouter` — not
`BrowserRouter` or `createBrowserRouter`. Hash routing avoids conflicts with the
platform's own routing.

`CssReset` normalizes browser styles. `CssVariables` injects the DHIS2 design tokens
(theme colors, spacers, elevations) as CSS custom properties so `@dhis2/ui` components
render correctly.

All routes are nested under the `SyncUrlWithGlobalShell` layout route, so every
page automatically keeps the Global Shell URL in sync. Add new routes as children
of that layout.

## Step 9: Create `src/interfaces/apiQueryTypes.ts`

```typescript
export type PossiblyDynamic<Type, InputType> = Type | ((input: InputType) => Type)
export type QueryVariables = Record<string, unknown>

type QueryParameterSingularValue = string | number | boolean
interface QueryParameterAliasedValue {
    [name: string]: QueryParameterSingularValue
}
type QueryParameterSingularOrAliasedValue = QueryParameterSingularValue | QueryParameterAliasedValue
type QueryParameterMultipleValue = QueryParameterSingularOrAliasedValue[]
export type QueryParameterValue =
    | QueryParameterSingularValue
    | QueryParameterAliasedValue
    | QueryParameterMultipleValue
    | undefined

export interface QueryParameters {
    pageSize?: number
    [key: string]: QueryParameterValue
}

export interface ResourceQuery {
    resource: string
    id?: PossiblyDynamic<string, QueryVariables>
    data?: PossiblyDynamic<unknown, QueryVariables>
    params?: PossiblyDynamic<QueryParameters, QueryVariables>
}
```

These types describe the shape of a DHIS2 API query passed to the data engine.
`ResourceQuery` is the main one — it maps to a DHIS2 API resource with optional
id, request body, and query parameters.

## Step 10: Create `src/utils/useApiDataQuery.ts`

```typescript
import { useDataEngine } from '@dhis2/app-runtime'
import {
    useQuery,
    QueryFunction,
    UseQueryOptions,
    QueryKey,
} from '@tanstack/react-query'
import { ResourceQuery } from '../interfaces/apiQueryTypes'

type UseApiDataQueryProps<
    TResultData,
    TError = Error,
    TData = TResultData,
    TQueryKey extends QueryKey = QueryKey,
> = Omit<UseQueryOptions<TResultData, TError, TData, TQueryKey>, 'queryFn'> & {
    query: ResourceQuery
}

export const useApiDataQuery = <
    TResultData,
    TError = Error,
    TData = TResultData,
    TQueryKey extends QueryKey = QueryKey,
>({
    query,
    queryKey,
    ...options
}: UseApiDataQueryProps<TResultData, TError, TData, TQueryKey>) => {
    const dataEngine = useDataEngine()

    const queryFn: QueryFunction<TResultData, TQueryKey> = async () => {
        const response = await dataEngine.query({ apiDataQuery: query })
        return response.apiDataQuery as TResultData
    }

    return useQuery<TResultData, TError, TData, TQueryKey>({
        queryKey,
        queryFn,
        ...options,
    })
}
```

Always use `useApiDataQuery` for data fetching — never use `useDataQuery` from
`@dhis2/app-runtime` directly.

## Step 11: Verify it runs

```bash
pnpm start
```

The app opens at `http://localhost:3000`. Confirm the login screen appears and you
can log in. For details on proxy mode, CORS handling, remote instances, platform
notes, and building for production, see `references/running-your-app.md`.

## After bootstrapping: ask about routing

Once the app is scaffolded and running, ask the user whether they'd like to set up
a multi-page application with sidebar navigation. Include this in your final response
after confirming the app runs — something like:

> "The app is up and running. Would you like me to set up routing with a sidebar
> layout? This gives you a collapsible sidebar for navigation, a page wrapper that
> keeps content readable on wide screens, and route-level control over layout. Most
> DHIS2 apps benefit from this — I can set it up now or you can add it later."

If the user says yes, read `references/routing.md` → `references/routing/sidebar.md`
and follow the implementation there.
