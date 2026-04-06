# Running Your DHIS2 App

A DHIS2 app needs a DHIS2 server to connect to and a way to handle CORS if the
server is remote. Unless the user has specified other credentials, default to
`admin` / `district` (the standard credentials for DHIS2 demo databases).

---

## Prerequisites

### DHIS2 instance

Check whether the user has DHIS2 running locally on the default port:

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/api/system/info
```

A DHIS2 instance will typically return **302** (redirect to login page) on unauthenticated
requests. If you get a 302, try the default credentials (`admin` / `district`):

```bash
curl -s -o /dev/null -w "%{http_code}" -u admin:district http://localhost:8080/api/system/info
```

- **200** — instance is running and credentials work. Use `http://localhost:8080` as the
  server URL. Default credentials indicate the standard Sierra Leone demo database.
- **Any other status (401, 403, connection error)** — either the credentials are wrong or
  nothing is running on port 8080. Ask the user which instance and credentials to use.
  Common setups:
  - A local instance on a non-default port
  - A remote server (e.g., a shared dev instance or DHIS2 play server)
  - A Docker container (often still on `localhost:8080`, but may differ)

### Node version

Node 18 or 20+ is required by the DHIS2 App Platform.

---

## Starting the dev server

### Local instance

If the DHIS2 server is on `localhost:8080`:

```bash
pnpm start
```

The app opens at `http://localhost:3000`. A login screen appears — enter
`http://localhost:8080` as the server URL and `admin` / `district` as credentials
(or whatever the user has specified).

### Remote instance (proxy mode)

If developing against a remote DHIS2 server, the browser will block API requests due to
CORS. The App Platform includes a built-in proxy to handle this:

```bash
pnpm start --proxy https://play.im.dhis2.org/dev
```

DHIS2 maintains play servers at `https://play.im.dhis2.org/<version>`. Use `/dev`
(master) by default. For a specific version, use the major version number — e.g.,
`/dev-2-43` for 2.43, `/dev-2-42` for 2.42. Only target a specific version if the
user requests it or the app needs to match a particular DHIS2 version.

The proxy starts on port 8080 and forwards API requests to the remote server, adding
the necessary CORS headers. At the login screen, enter `http://localhost:8080` as the
server URL (not the remote URL) and `admin` / `district` as credentials (or
whatever the user has specified).

---

## Platform notes (App Platform v12)

- **File extensions** — Vite requires `.tsx` for JSX files. Plain `.ts` cannot contain JSX.
- **Environment variables** — use the `DHIS2_` prefix (not `REACT_APP_`).
- **Globals** — use `window.variableName`, not `global.variableName`.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| CORS errors in the browser console | Developing against a remote instance without the proxy | Use `pnpm start --proxy <url>` instead of plain `pnpm start` |
| Login screen rejects credentials | Wrong server URL entered at login | Use `http://localhost:8080` when running with `--proxy`, or the actual instance URL for local instances |
| `http://localhost:3000` shows a blank page or connection refused | Dev server not running or port conflict | Check terminal for errors. Kill any process on port 3000 and restart with `pnpm start` |
| `ECONNREFUSED` when starting with `--proxy` | Remote instance URL is wrong or unreachable | Verify the URL works in a browser first. Check for VPN requirements |
| App loads but API calls return 401/403 | Session expired or insufficient permissions | Log out and back in. Verify the user account has the required authorities on that DHIS2 instance |
