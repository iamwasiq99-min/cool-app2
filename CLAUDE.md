# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A React + TypeScript + Vite frontend backed by an AWS Amplify Gen 2 backend. The Amplify backend (`amplify/`) and the Vite frontend (`src/`) are separate TypeScript projects with their own `package.json`/`tsconfig.json`.

## Commands

Run from the repo root unless noted otherwise.

- `npm run dev` — start the Vite dev server
- `npm run build` — type-check (`tsc -b`) then production build via Vite
- `npm run lint` — run ESLint over the whole repo
- `npm run preview` — preview the production build locally

There is no test suite configured in this repo.

Amplify backend (sandbox / cloud deploy) commands are run via `@aws-amplify/backend-cli`, e.g. `npx ampx sandbox` to spin up a personal cloud sandbox backend while developing. There's no npm script wrapper for this — invoke `ampx` directly.

## Architecture

- `amplify/backend.ts` is the single entry point that wires together backend resources via `defineBackend({ auth, data })`. Add new resources (storage, functions, etc.) here.
- `amplify/auth/resource.ts` defines Cognito auth (`defineAuth`) — currently email-based login only.
- `amplify/data/resource.ts` defines the data layer (`defineData`) — an AppSync GraphQL API generated from the `a.schema(...)` model definitions, currently a single `Todo` model with guest (unauthenticated) access via `identityPool` as the default authorization mode.
- The frontend (`src/`) talks to the backend through `aws-amplify`'s generated Data client (`generateClient<Schema>()`), using the `Schema` type exported from `amplify/data/resource.ts`. Frontend code does not currently call this client yet — `src/App.tsx` is still the default Vite template.
- `amplify/` has its own `tsconfig.json` and `package.json` (`"type": "module"`) distinct from the root ones — keep backend-only dependencies (`@aws-amplify/backend`, `aws-cdk-lib`, `constructs`, etc.) in mind as belonging to that project even though there's a single root `node_modules`.

## Notes

- `.gitignore` has unusually broad entries (`auth`, `backend.ts`, `data`, `package.json`, `tsconfig.json` with no path prefix) intended to ignore local Amplify sandbox artifacts, but the real `amplify/auth`, `amplify/backend.ts`, `amplify/data`, `amplify/package.json`, and `amplify/tsconfig.json` are already tracked in git so this has no effect on them. Be aware any *new* untracked file with one of these exact names/paths elsewhere in the repo will be silently ignored.
