---
name: dhis2-app-dev
description: >
  Guide for building DHIS2 custom applications using the DHIS2 App Platform.
  Use this skill whenever the user wants to create, scaffold, or bootstrap a DHIS2 app,
  or mentions DHIS2, d2, dhis2 app platform, or any DHIS2-specific terminology —
  even if they don't explicitly say "DHIS2 app."
---

# DHIS2 Application Development

You are helping a developer build a custom DHIS2 web application. DHIS2 is a health information
management platform with its own app framework, component library, and Web API. AI assistants
frequently get DHIS2 wrong — using deprecated API endpoints, incorrect patterns, or generic
libraries instead of the DHIS2-specific tooling. This skill exists to prevent that.

## Scaffolding a New App

Always scaffold with the official CLI — never create the project structure manually:

```bash
pnpm create @dhis2/app@latest <app-name> --yes
```

The `--yes` flag uses sensible defaults (pnpm, JavaScript, basic template). This generates a
working app with `@dhis2/app-runtime` and the DHIS2 build tooling already configured.

After scaffolding:

```bash
cd <app-name>
pnpm install
pnpm start --proxy <dhis2-instance-url>
```

The `--proxy` flag routes API requests through a local proxy to avoid CORS issues when developing
against a remote DHIS2 instance. At the login screen, enter `http://localhost:8080` as the
server URL.

For project structure details, d2.config.js options, template choices, and the full development
workflow, read `references/bootstrapping.md`.
