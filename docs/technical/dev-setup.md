# Development Setup

## Snapshot

- Snapshot date: **April 23, 2026**.
- Scope: onboarding from repository clone to opening the first pull request.
- Evidence policy: all steps are derived from repository scripts, configs, and workflows.

## Goal

- Get a local development environment running.
- Make one small change safely.
- Open a PR that passes repository automation.

## 1) Prerequisites

- Git installed.
- Node.js installed. Minimum supported version is `>=18.0.0`.
- Node `20.x` is recommended to match GitHub Actions runtime.
- Yarn Classic installed (`yarn@1.22.22`).
- Optional: Docker + Docker Compose for containerized run.

## 2) Clone The Repository

```bash
git clone <your-repo-url>
cd 2026-fwdays-agentic-large-day01-hw
```

Optional if using a fork workflow:

```bash
git remote add upstream <upstream-repo-url>
git fetch upstream
```

## 3) Install Dependencies

```bash
yarn install
```

Notes:

- The monorepo uses Yarn workspaces (`excalidraw-app`, `packages/*`, `examples/*`).
- Root `package.json` defines `packageManager: yarn@1.22.22`.
- Use Yarn for consistency with scripts and lockfile behavior.

If your local dependency tree is broken:

```bash
yarn clean-install
```

## 4) Configure Environment

- Vite env files are stored at repository root (`.env.development`, `.env.production`).
- Local overrides should go into local env files (for example `.env.local`, `.env.development.local`).
- Development app port is controlled by `VITE_APP_PORT` (default in repo: `3001`).

Important service endpoints in development env:

- `VITE_APP_WS_SERVER_URL=http://localhost:3002` for collaboration websocket.
- `VITE_APP_AI_BACKEND=http://localhost:3016` for AI-related features.
- Firebase and share backend URLs are already defined in tracked env files.

## 5) Run The App Locally

From repository root:

```bash
yarn start
```

What this does:

- Delegates to `excalidraw-app` workspace.
- Runs `vite` dev server.
- Opens browser automatically.

Expected URL:

- `http://localhost:3001` (unless `VITE_APP_PORT` is overridden).

## 6) Validate Your Setup

Quick checks:

```bash
yarn test:other
yarn test:code
yarn test:typecheck
```

Full local check before PR:

```bash
yarn test:all
```

Optional coverage run:

```bash
yarn test:coverage
```

## 7) Create Your First Change

Create a branch:

```bash
git checkout -b docs/dev-setup-onboarding
```

Make your change, then run at least the relevant checks:

```bash
yarn test:other
```

For code changes, run full quality checks:

```bash
yarn test:all
```

Commit:

```bash
git add <files>
git commit -m "docs: add dev setup guide"
```

Note on local hooks:

- `.husky/pre-commit` currently does not execute lint-staged commands.
- Do not rely on hooks for validation; run checks manually.

## 8) Push And Open The First PR

Push your branch:

```bash
git push -u origin docs/dev-setup-onboarding
```

Open a PR in GitHub:

- Target branch for upstream contribution is typically `master` in this repo.
- PR title must be semantic because CI runs `action-semantic-pull-request`.
- Example valid title: `docs: add dev setup onboarding`.
- Fill the PR template checklist in `.github/PULL_REQUEST_TEMPLATE.md`.

## 9) CI Checks You Should Expect

On pull requests, repository workflows run checks including:

- Semantic PR title validation.
- Lint + format + typecheck (`yarn test:other`, `yarn test:code`, `yarn test:typecheck`).
- Coverage run (`yarn test:coverage`).
- Bundle size check for `@excalidraw/excalidraw` on PRs targeting `master`.

## 10) Optional Workflows

Build production app locally:

```bash
yarn build
```

Run production build locally:

```bash
yarn start:production
```

Build local packages:

```bash
yarn build:packages
```

Run browser-script integration example:

```bash
yarn start:example
```

Run via Docker:

```bash
docker compose up --build
```

Expected Docker URL:

- `http://localhost:3000`

## 11) Troubleshooting

- If `yarn` version mismatches, install `yarn@1.22.22`.
- If port is busy, change `VITE_APP_PORT` in local env override file.
- If collaboration features fail locally, ensure websocket server exists at `VITE_APP_WS_SERVER_URL`.
- If AI features fail locally, ensure backend exists at `VITE_APP_AI_BACKEND`.
- If dependencies become inconsistent, run `yarn clean-install`.

## Source Verification

- `package.json`
- `excalidraw-app/package.json`
- `excalidraw-app/vite.config.mts`
- `.env.development`
- `.env.production`
- `.husky/pre-commit`
- `.lintstagedrc.js`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/workflows/semantic-pr-title.yml`
- `.github/workflows/lint.yml`
- `.github/workflows/test-coverage-pr.yml`
- `.github/workflows/size-limit.yml`
- `docker-compose.yml`
- `Dockerfile`
