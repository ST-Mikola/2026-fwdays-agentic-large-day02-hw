# Active Context

## Snapshot Date

- Context snapshot date: **May 11, 2026**.
- Scope of this file: current working focus for this repository and the Memory Bank baseline.

## Current Focus

- Keep the Excalidraw Memory Bank synchronized with the real codebase structure and flows.
- Keep repo-local Cursor rules and skills aligned with documented maintenance workflows.
- Keep the expanded `.cursor/rules` set aligned with real repository workflows for locales, SCSS, components, and collaboration.
- Treat `excalidraw-app` as the primary product surface and `packages/*` as the engine/platform layer.
- Preserve high confidence in collaboration, persistence, and sharing flows, because these are the highest UX-risk paths.
- Document decisions and hotspots in `docs/memory/*` so future work starts with verified context, not assumptions.

## What Is Already Completed

- `projectbrief.md` created and aligned with workspace/package structure.
- `techContext.md` created with pinned versions, runtime assumptions, and script commands.
- `systemPatterns.md` created with architecture and implementation patterns (state, sync, persistence, build).
- `docs/technical/architecture.md` created and validated against current source code.
- `productContext.md` created with UX goals and key user scenarios.
- `progress.md` created to track Memory Bank completion state and update triggers.
- `decisionLog.md` created to capture durable architectural and product-flow decisions inferred from the codebase.
- Added `.cursor/rules/memory-bank.mdc` and aligned the `memory-bank-update` skill so Memory Bank activation is explicit and the workflow stays in one place.
- Expanded `.cursor/rules` to cover locales, SCSS token usage, SCSS nesting, component maintenance, and a module-specific collaboration rule.
- These files are now internally aligned around the same repo model: monorepo app shell + reusable editor engine + collaboration/persistence integrations.

## Immediate Working Priorities

- Finalize and maintain this Memory Bank set as the source of operational context.
- Keep `.cursor/rules/memory-bank.mdc` and `.cursor/skills/memory-bank-update/SKILL.md` in sync when the maintenance workflow changes.
- Keep `.cursor/rules/locales.mdc`, `.cursor/rules/scss-*.mdc`, `.cursor/rules/components.mdc`, and `.cursor/rules/collab-module.mdc` aligned with actual repo conventions.
- Keep bonus Memory Bank files focused on current work instead of repeating project-wide facts from other docs.
- Keep architecture and product documentation tied to concrete files when code changes.
- Prioritize review of changes affecting collaboration session lifecycle (`collab/`).
- Prioritize review of changes affecting local/browser persistence (`data/LocalData.ts`, `data/tabSync.ts`).
- Prioritize review of changes affecting share/export reliability (`data/index.ts`, `share/ShareDialog.tsx`).
- Prioritize review of changes affecting remote sync reconciliation (`packages/excalidraw/data/reconcile.ts`).

## What Is Not Yet Covered

- `docs/product/domain-glossary.md` is present.
- `docs/product/PRD.md` is present.

## Active Hotspots In Code

- Collaboration control plane hotspot: `excalidraw-app/collab/Collab.tsx` and `excalidraw-app/collab/Portal.tsx`.
- Scene/file persistence hotspot: `excalidraw-app/data/LocalData.ts`, `excalidraw-app/data/firebase.ts`, `excalidraw-app/data/FileManager.ts`.
- App orchestration and UX composition hotspot: `excalidraw-app/App.tsx`, `excalidraw-app/share/ShareDialog.tsx`, `excalidraw-app/components/AppMainMenu.tsx`, `excalidraw-app/components/AppWelcomeScreen.tsx`.
- Editor reconciliation and history boundary hotspot: `packages/excalidraw/data/reconcile.ts` and `packages/excalidraw/components/App.tsx`.

## Current Risks To Watch

- Drift between docs and code when app-level flows evolve quickly.
- Regressions in remote-update handling versus undo/redo expectations (`captureUpdate` boundaries).
- Edge cases in offline/online transitions during active collaboration sessions.
- Persistence inconsistencies across tabs if version signaling or restore paths are modified.
- Export/share reliability when image file status is still pending at export time.

## Near-Term “Done” Criteria For This Context

- `docs/memory/activeContext.md` exists and stays under 200 lines.
- The file reflects actual repository state on May 11, 2026.
- Focus areas map to concrete source files and match existing Memory Bank docs.
- No contradictory statements across `projectbrief.md`, `techContext.md`, `systemPatterns.md`, `productContext.md`, and `activeContext.md`.

## Details

For detailed architecture → see [architecture.md](../technical/architecture.md)  
For undocumented runtime contracts → see [undocumented-behaviors.md](../technical/undocumented-behaviors.md)  
For product requirements → see [PRD.md](../product/PRD.md)  
For domain glossary → see [domain-glossary.md](../product/domain-glossary.md)
