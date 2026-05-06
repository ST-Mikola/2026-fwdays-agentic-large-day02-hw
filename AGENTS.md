# AGENTS.md

## Project Structure

Excalidraw is a **monorepo** with a clear separation between the core library and the application:

- **`packages/excalidraw/`** - Main React component library published to npm as `@excalidraw/excalidraw`
- **`excalidraw-app/`** - Full-featured web application (excalidraw.com) that uses the library
- **`packages/`** - Core packages: `@excalidraw/common`, `@excalidraw/element`, `@excalidraw/math`, `@excalidraw/utils`
- **`examples/`** - Integration examples (NextJS, browser script)

## Development Workflow

1. **Package Development**: Work in `packages/*` for editor features
2. **App Development**: Work in `excalidraw-app/` for app-specific features
3. **Testing**: Always run `yarn test:update` before committing
4. **Type Safety**: Use `yarn test:typecheck` to verify TypeScript

## Development Commands

```bash
yarn test:typecheck  # TypeScript type checking
yarn test:update     # Run all tests (with snapshot updates)
yarn fix             # Auto-fix formatting and linting issues
```

## Architecture Notes

### Package System

- Uses Yarn workspaces for monorepo management
- Internal packages use path aliases (see `vitest.config.mts`)
- Build system uses esbuild for packages, Vite for the app
- TypeScript throughout with strict configuration

# Memory bank

Use the memory bank in `docs/memory/` to keep durable project context up to date.

## Memory Bank Pattern

- `docs/memory/projectbrief.md` - project goals, scope, and constraints
- `docs/memory/productContext.md` - user needs, workflows, and product context
- `docs/memory/systemPatterns.md` - architecture, conventions, and recurring design patterns
- `docs/memory/techContext.md` - stack, tooling, environment details, and technical constraints
- `docs/memory/activeContext.md` - current focus, active tasks, and short-term decisions
- `docs/memory/progress.md` - completed work, remaining work, and current status
- `docs/memory/decisionLog.md` - notable decisions, rationale, and tradeoffs

## Memory Bank Rules

1. Read the relevant files in `docs/memory/` before making substantial project changes.
2. Update the memory bank after each project change so the documented state matches the codebase.
3. Record durable information in the appropriate file and keep transient task state in `activeContext.md`.
4. Add decision entries to `decisionLog.md` when a change introduces a meaningful architectural, product, or workflow choice.
5. Keep entries concise, factual, and scoped to information that will help future work.
