# Decision Log

## Snapshot

- Snapshot date: **April 23, 2026**.
- Scope: architectural and product-level decisions observable in the current codebase and Memory Bank baseline.
- Note: these entries are inferred from implementation and current structure, not from a formal ADR directory.

## Decision Entries

### D-001: Monorepo With Workspace-Coupled App + Packages

- Status: Accepted.
- Decision: keep `excalidraw-app` and `packages/*` in one workspace-managed repository.
- Why: enables synchronized evolution of product UX and package API without publish-loop friction.
- Consequences: local source aliasing is required for app/test tooling; build scripts coordinate app and package pipelines from root.
- Evidence: root `package.json`, `tsconfig.json`, `vitest.config.mts`.

### D-002: Product Shell Consumes Reusable Editor Engine

- Status: Accepted.
- Decision: app shell composes around `<Excalidraw>` instead of forking engine internals into app layer.
- Why: clear separation of product concerns (menus/share/collab UX) vs editor core concerns (scene/render/actions).
- Consequences: app customization happens through props/wrappers/companion components; editor behavior remains centralized in `packages/excalidraw`.
- Evidence: `excalidraw-app/App.tsx`, `packages/excalidraw/index.tsx`, `packages/excalidraw/components/App.tsx`.

### D-003: Isolated State Stores For App And Engine

- Status: Accepted.
- Decision: maintain separate Jotai stores/scopes for app-level and editor-level state.
- Why: reduce accidental cross-layer coupling and preserve clearer ownership boundaries.
- Consequences: app state atoms live in `excalidraw-app/app-jotai.ts`; editor state uses isolated provider in `packages/excalidraw/editor-jotai.ts`.
- Evidence: `excalidraw-app/app-jotai.ts`, `packages/excalidraw/editor-jotai.ts`, `excalidraw-app/App.tsx`.

### D-004: Collaboration Over Socket Transport + Client-Side Encryption

- Status: Accepted.
- Decision: collaboration events flow through socket transport while payloads are encrypted client-side.
- Why: real-time UX with privacy-preserving payload handling and shareable room links.
- Consequences: collaboration startup/join logic is concentrated in `Collab`; socket emission/reception and encryption boundary is concentrated in `Portal`.
- Evidence: `excalidraw-app/collab/Collab.tsx`, `excalidraw-app/collab/Portal.tsx`, `excalidraw-app/data/index.ts`.

### D-005: Deterministic Remote Reconciliation Policy

- Status: Accepted.
- Decision: reconcile remote and local elements with deterministic conflict rules and index normalization.
- Why: avoid divergent scenes and reduce nondeterministic merge outcomes in collaboration.
- Consequences: local editing/newer versions can win over remote updates; invalid fractional indices are repaired after merge.
- Evidence: `packages/excalidraw/data/reconcile.ts`, `excalidraw-app/collab/Collab.tsx`.

### D-006: Remote Updates Must Not Pollute Local Undo Stack

- Status: Accepted.
- Decision: apply collaboration/restore-driven scene updates with `CaptureUpdateAction.NEVER`.
- Why: preserve intuitive undo/redo for user-authored local edits.
- Consequences: explicit capture-mode discipline is required across update paths.
- Evidence: `excalidraw-app/collab/Collab.tsx`, `excalidraw-app/App.tsx`, `packages/excalidraw/components/App.tsx`.

### D-007: Local-First Persistence With Multi-Tier Storage

- Status: Accepted.
- Decision: persist scene/app state in `localStorage` and binary files in IndexedDB.
- Why: resilience and quick resume without mandatory network dependency.
- Consequences: save operations are debounced and lock-aware; file lifecycle needs explicit status tracking and cleanup paths.
- Evidence: `excalidraw-app/data/LocalData.ts`, `excalidraw-app/data/FileManager.ts`, `excalidraw-app/data/localStorage.ts`.

### D-008: Lightweight Multi-Tab Freshness Signaling

- Status: Accepted.
- Decision: use storage-backed version timestamps to detect when another tab has fresher state.
- Why: simple cross-tab coherence without introducing a dedicated sync backend.
- Consequences: restore/sync logic must check version keys before applying browser state.
- Evidence: `excalidraw-app/data/tabSync.ts`, `excalidraw-app/App.tsx`.

### D-009: Firebase As Persistence Backend For Shared Scenes/Files

- Status: Accepted.
- Decision: use Firestore for scene documents and Firebase Storage for file blobs.
- Why: persistent shared-state backing for collaboration and export-related flows.
- Consequences: transactional scene writes and file upload/download handling are required; runtime depends on valid `VITE_APP_FIREBASE_CONFIG`.
- Evidence: `excalidraw-app/data/firebase.ts`, `firebase-project/firestore.rules`, `firebase-project/storage.rules`.

### D-010: Sharing UX Supports Two Modes (Live Collab vs Link Export)

- Status: Accepted.
- Decision: keep collaboration-room sharing and static link export as separate but adjacent user flows.
- Why: users need both synchronous co-editing and asynchronous sharing.
- Consequences: share dialog contains picker/state for both modes; export errors and collaboration errors use distinct feedback paths.
- Evidence: `excalidraw-app/share/ShareDialog.tsx`, `excalidraw-app/App.tsx`, `packages/excalidraw/components/ShareableLinkDialog.tsx`.

### D-011: Error Recovery Uses Top-Level Boundary + Telemetry

- Status: Accepted.
- Decision: route uncaught UI errors through a top boundary that captures telemetry and provides recovery actions.
- Why: maintain usability during crash states and improve triage quality.
- Consequences: users can clear local storage and reload; crash context can be routed to issue reporting flow.
- Evidence: `excalidraw-app/components/TopErrorBoundary.tsx`.

### D-012: Progressive Enhancement For Optional Capabilities

- Status: Accepted.
- Decision: load optional capabilities opportunistically (PWA prompt, lazy collaboration deps, AI flows).
- Why: preserve baseline startup experience while enabling richer workflows when available.
- Consequences: optional services are guarded by runtime checks and env-backed endpoints.
- Evidence: `excalidraw-app/index.tsx`, `excalidraw-app/App.tsx`, `excalidraw-app/components/AI.tsx`.

## Undocumented Behavior Entries

These entries do not describe new product decisions. They capture fragile implementation contracts where current code behavior is more specific than the current architecture/product docs.

### UB-001: Hybrid Double-Tap / Double-Click State Machine

- Status: Observed in implementation.
- What code does: editor click handling mixes browser events with module-level mutable flags (`didTapTwice`, `firstTapPosition`, `tappedTwiceTimer`) to infer touch and double-activation intent across event boundaries.
- What is documented: current technical docs describe editor event handling at a high level, but do not describe this implicit cross-event state machine or its mobile-text-editing dependency.
- Gap / consequence: refactors that treat click handling as a simpler browser-native flow can break touch insertion and double-click semantics.
- Evidence: `packages/excalidraw/components/App.tsx`; see also `docs/technical/undocumented-behaviors.md` behavior #1 and #3.

### UB-002: Capture Scheduling Has Hidden Precedence Rules

- Status: Observed in implementation.
- What code does: `CaptureUpdateAction` scheduling behaves like an implicit priority system (`IMMEDIATELY > NEVER > EVENTUALLY`), and the scheduling API is acknowledged in code comments as error-prone across many call sites.
- What is documented: `architecture.md` and this log explain that remote updates use `CaptureUpdateAction.NEVER`, but they do not spell out precedence rules or the scheduling invariants required to keep history/increments correct.
- Gap / consequence: seemingly safe cleanup of capture scheduling can corrupt undo/redo behavior, increment emission, and sync semantics.
- Evidence: `packages/element/src/store.ts`; see also `docs/technical/undocumented-behaviors.md` behavior #6.

### UB-003: Collaboration Startup Uses Multi-Path Initialization

- Status: Observed in implementation.
- What code does: collaboration bootstrapping can initialize through socket `INIT`, connect-error fallback, or timeout fallback, and these paths share an overloaded scene-promise contract.
- What is documented: collaboration is documented as a room-based real-time flow, but the explicit precedence/race behavior of startup paths is not documented in product or architecture docs.
- Gap / consequence: simplifying startup logic without characterization tests can introduce duplicate initialization, dropped initial scenes, or race regressions.
- Evidence: `excalidraw-app/collab/Collab.tsx`; see also `docs/technical/undocumented-behaviors.md` behavior #12.

### UB-004: `editor:initialize` Fires After Async Loading, Not On Mount

- Status: Observed in implementation.
- What code does: `editor:initialize` and `onInitialize` fire from `componentDidUpdate` only after `isLoading` becomes `false`, rather than directly during mount.
- What is documented: docs describe async scene restoration and editor bootstrapping, but do not explicitly state that API readiness is delayed until after loading completes.
- Gap / consequence: integrations may incorrectly assume mount-time readiness and break if lifecycle code is reorganized without preserving this contract.
- Evidence: `packages/excalidraw/components/App.tsx`; see also `docs/technical/undocumented-behaviors.md` behavior #15.

## Open Decisions

- No additional unresolved architecture decisions were explicitly recorded in code comments as formal ADRs.
- If ADR discipline is introduced later, this file should map one-to-one with ADR IDs and links.

## Maintenance Note

- Add a new decision entry when a code change introduces a durable architecture, product-flow, or operational policy change.
- Update an existing entry when the implementation still follows the same decision but the reasoning or consequences have shifted.
- Replace inferred entries with links to formal ADRs if the repository adopts ADRs later.

## Details

For detailed architecture → see [architecture.md](../technical/architecture.md)  
For undocumented runtime contracts → see [undocumented-behaviors.md](../technical/undocumented-behaviors.md)  
For product requirements → see [PRD.md](../product/PRD.md)  
For domain glossary → see [domain-glossary.md](../product/domain-glossary.md)
