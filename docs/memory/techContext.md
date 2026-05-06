# Tech Context

## Runtime And Package Management

- Monorepo name: `excalidraw-monorepo`.
- Package manager: `yarn@1.22.22` (declared in root `package.json`).
- Workspace layout: `excalidraw-app`, `packages/*`, `examples/*`.
- Node.js engine requirement: `>=18.0.0` (root and app package manifests).
- Container build runtime: `node:18` (Docker build stage).
- Container serving runtime: `nginx:1.27-alpine` (Docker runtime stage).

## Core Frontend Stack

- Language: TypeScript (`typescript` pinned to `5.9.3` at root).
- UI framework: React `19.0.0` and React DOM `19.0.0` (app and examples).
- Main bundler/dev server: Vite `5.0.12`.
- Main React plugin: `@vitejs/plugin-react` `3.1.0`.
- Editor UI atoms use Jotai `2.11.0` in the app and package layer.
- Realtime client: `socket.io-client` `4.7.2`.
- Persistence backend SDK: Firebase `11.3.1` (Firestore and Storage are used in the app data layer).
- Error monitoring client: `@sentry/browser` `9.0.1`.
- Styling and UI utilities in the editor package include Sass `1.51.0`, `clsx` `1.1.1`, and `radix-ui` `1.4.3`.

## Excalidraw Package Layer

- Main library package: `@excalidraw/excalidraw` version `0.18.0`.
- Supporting local packages: `@excalidraw/common`, `@excalidraw/element`, `@excalidraw/math`, `@excalidraw/utils`.
- Utility package in repo: `@excalidraw/utils` version `0.1.2`.
- `@excalidraw/excalidraw` peer dependency range supports React 17/18/19.
- Package exports include environment-aware `development`/`production` entries.
- The repository also contains integration examples for Next.js and direct browser-script usage.

## Testing, Linting, Formatting

- Unit/integration test runner: Vitest `3.0.6`.
- Test environment: `jsdom` (from `vitest.config.mts`).
- Coverage engine/plugin: `@vitest/coverage-v8` `3.0.7`.
- Linting: ESLint with `@excalidraw/eslint-config` + `eslint-config-react-app`.
- Formatting: Prettier `2.6.2` with shared config `@excalidraw/prettier-config`.
- Git hooks: Husky `7.0.4` + lint-staged `12.3.7`.

## TypeScript And Local Linking Model

- TS mode: strict (`strict: true`) with `noEmit: true` at root.
- Module target: `ESNext`, JSX mode: `react-jsx`.
- Root `tsconfig` maps `@excalidraw/*` imports directly to local `packages/*/src`.
- `vitest.config.mts` defines matching alias mapping for test execution.
- This setup enables app/package co-development without publishing intermediate package versions.

## Build And Delivery

- Root build entrypoint: `yarn build` (delegates to `excalidraw-app` build).
- Package build flow: `yarn build:packages` builds common/math/element/excalidraw.
- Docker image build runs `yarn build:app:docker` and serves static build via Nginx.
- `docker-compose.yml` maps container port `80` to host `3000`.
- Vercel deployment config is present in `vercel.json`.

## Common Commands (From Source Scripts)

### Monorepo Root

- Install dependencies: `yarn install`.
- Start app (delegated): `yarn start`.
- Production start flow: `yarn start:production`.
- Build app: `yarn build`.
- Build packages only: `yarn build:packages`.
- Start browser-script example after package build: `yarn start:example`.
- Run all checks: `yarn test:all`.
- Run app tests: `yarn test:app`.
- Run type-check: `yarn test:typecheck`.
- Run ESLint: `yarn test:code`.
- Check formatting: `yarn test:other`.
- Run coverage: `yarn test:coverage`.
- Auto-fix lint + format: `yarn fix`.
- Clean build artifacts: `yarn rm:build`.
- Full reinstall: `yarn clean-install`.

### App Workspace (`excalidraw-app`)

- Dev server: `yarn start` (runs `yarn && vite`).
- Docker-oriented app build: `yarn build:app:docker`.
- Production static serve: `yarn start:production`.
- Serve built app manually: `yarn serve` (http-server on `localhost:5001`).
- Preview built app: `yarn build:preview` (Vite preview on `5000`).

### Example Workspaces

- Next.js example dev: `yarn --cwd examples/with-nextjs dev` (port `3005`).
- Next.js example start: `yarn --cwd examples/with-nextjs start` (port `3006`).
- Script-in-browser example dev: `yarn --cwd examples/with-script-in-browser start`.

### Docker

- Compose up: `docker compose up --build`.
- Service name: `excalidraw`.
- Published host URL (default compose): `http://localhost:3000`.

## Environment And Config Notes

- App reads Firebase config from `import.meta.env.VITE_APP_FIREBASE_CONFIG`.
- Environment templates exist: `.env.development`, `.env.production`.
- Build scripts inject Vite flags such as `VITE_APP_DISABLE_SENTRY` and `VITE_APP_ENABLE_TRACKING`.

## Local Agent Tooling

- Repo-local Cursor rules live under `.cursor/rules/*.mdc`.
- Repo-local Cursor skills live under `.cursor/skills/*/SKILL.md`.
- Memory Bank maintenance is activated by `.cursor/rules/memory-bank.mdc` and implemented by `.cursor/skills/memory-bank-update/SKILL.md`.

## Details

For detailed architecture → see [architecture.md](../technical/architecture.md)  
For undocumented runtime contracts → see [undocumented-behaviors.md](../technical/undocumented-behaviors.md)  
For product requirements → see [PRD.md](../product/PRD.md)  
For domain glossary → see [domain-glossary.md](../product/domain-glossary.md)
