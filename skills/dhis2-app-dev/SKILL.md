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

## First: determine where you are

Before doing anything, figure out whether you're in an existing DHIS2 project or starting fresh.

| Check | How | Result |
|-------|-----|--------|
| `d2.config.js` exists in project root | Glob for `d2.config.js` | **Existing project** — skip scaffolding |
| `@dhis2/app-runtime` in package.json | Read `package.json` | **Existing project** — skip scaffolding |
| Neither found | — | **New project** — follow `references/bootstrapping.md` |

If it's an existing project, read `d2.config.js` and `package.json` to understand what's
already configured before making changes. Then jump to the relevant section below for whatever
the user needs.

If it's a new project, follow the full bootstrapping workflow in `references/bootstrapping.md`.
That guide walks through scaffolding, installing the tech stack, and configuring everything
step by step.
