# System Patterns

## High-Level Architecture

- The repository follows a monorepo workspace model with three main runtime zones: `excalidraw-app`, `packages/*`, `examples/*`.
- `excalidraw-app` acts as the product shell (menus, collaboration UX, persistence wiring, app-specific integrations).
- `packages/excalidraw` acts as the reusable editor engine exposed as a React component.
- Domain packages (`@excalidraw/common`, `@excalidraw/element`, `@excalidraw/math`, `@excalidraw/utils`) provide lower-level primitives.
- Examples are consumer-facing integration references, not core runtime dependencies of the app.

## Layering Pattern

- Product app imports the editor package (`@excalidraw/excalidraw`) instead of duplicating editor internals.
- Shared package imports are resolved to local sources during development via root `tsconfig.json` path aliases.
- Test runtime follows the same alias mapping through `vitest.config.mts`.
- This creates a package-first architecture where app and packages evolve together in one codebase.

## App Shell + Engine Composition

- `excalidraw-app/App.tsx` composes app-level UI around the `<Excalidraw>` engine component.
- App shell owns collaboration controls and room lifecycle.
- App shell owns import/export workflows.
- App shell owns language and theme integration.
- App shell owns app-specific menus and dialogs.
- Engine responsibilities stay in `packages/excalidraw` (canvas, scene model, actions, editor behavior).

## Core Editor State Pattern

- The main editor is centered around the class component `packages/excalidraw/components/App.tsx`.
- State is intentionally split between React `AppState` and non-React domain objects.
- React `AppState` stores editor UI/session state such as active tool, selection ids, dialogs, zoom, scroll, theme, and viewport size.
- `Scene` is the source of truth for elements and element maps, including deleted-element history and cached selected-element queries.
- `Store` captures observed changes and emits durable or ephemeral increments.
- `History` subscribes to durable store increments and turns them into undo/redo entries.
- This means Excalidraw does not keep the full scene graph in React state; React coordinates the UI while `Scene` owns the element model.

## Action System Pattern

- User commands are modeled as registered actions under `packages/excalidraw/actions/*`.
- `ActionManager` is the dispatch layer for keyboard shortcuts, toolbar/menu actions, and programmatic action execution.
- Each action receives `(elements, appState, value, app)` and returns an `ActionResult`.
- `App.syncActionResult(...)` is the central reducer-like bridge that applies action output to `Scene`, files/image cache, and React state.
- This pattern keeps editor behaviors modular while still routing updates through one integration point.

## Rendering Pipeline Pattern

- Rendering is split between React UI composition and imperative canvas rendering.
- `Renderer.getRenderableElements(...)` derives visible/renderable elements from `Scene` using viewport state and a scene nonce.
- `StaticCanvas`, `InteractiveCanvas`, and `NewElementCanvas` render different layers of the editor instead of a single monolithic canvas pass.
- `Scene` updates trigger render invalidation, while the renderer memoizes viewport-dependent element filtering.
- The pipeline is optimized for large scenes by filtering to visible elements before drawing.

## State Isolation Pattern (Jotai)

- App-level state uses `appJotaiStore` (`excalidraw-app/app-jotai.ts`).
- Editor-level state uses isolated Jotai scope (`packages/excalidraw/editor-jotai.ts` via `createIsolation()`).
- App tree wraps root in both app/provider contexts (`Provider`, `ExcalidrawAPIProvider`), while editor internals are wrapped by `EditorJotaiProvider`.
- This separation limits accidental coupling between product UI state and editor engine state.

## Collaboration Architecture Pattern

- `Collab` class orchestrates collaboration session lifecycle and high-level sync policies.
- `Portal` class encapsulates socket transport concerns (open/close, emit/listen, encrypted payload broadcast).
- Collaboration startup lazily imports `socket.io-client`, then joins/creates a room.
- A fallback initialization path handles socket connect failures and delayed first scene events.
- Room identity is URL-based (`#room=<roomId>,<roomKey>`), enabling shareable collaboration links.

## Event-Driven Sync Pattern

- Collaboration messages are typed by `WS_SUBTYPES` and routed by switch-based handlers.
- Distinct message classes exist for scene init, scene updates, pointer state, idle state, and visible bounds.
- Transport events are split into regular and volatile channels (`server-broadcast` vs `server-volatile-broadcast`).
- User-follow and viewport sync are modeled as app state updates triggered by websocket events.

## Deterministic Reconciliation Pattern

- Remote scene data is reconciled through `reconcileElements(...)` from the core package.
- Conflict strategy prefers local element during active local editing, newer local version, or deterministic `versionNonce` tie-break.
- Reconciliation normalizes ordering with fractional index utilities and repairs invalid indices.
- Collaboration layer additionally bumps element versions and records last broadcast/received scene version to avoid echo loops.

## Undo/History Boundary Pattern

- Scene updates carry explicit `captureUpdate` semantics (`IMMEDIATELY`, `EVENTUALLY`, `NEVER`).
- Remote/collab-driven updates use `CaptureUpdateAction.NEVER` to avoid polluting local undo history.
- Local user actions use immediate/eventual capture depending on action type.
- `Store.commit(...)` is the point where scheduled updates become emitted increments and can be recorded by `History`.

## Local-First Persistence Pattern

- `LocalData` persists app state + elements to `localStorage` and files to IndexedDB (`idb-keyval`).
- Saves are debounced and can be paused/resumed via a lock (`Locker`) to prevent race conditions (for example during collaboration).
- File persistence is separated from scene persistence but coordinated in one save flow.
- Storage quota issues are surfaced via an atom (`localStorageQuotaExceededAtom`).

## Multi-Tab Coherence Pattern

- `tabSync.ts` keeps version timestamps in localStorage for data/files state.
- Each tab tracks in-memory version markers and can detect when browser storage became newer.
- This provides lightweight cross-tab state freshness signaling without a dedicated sync service.

## File Lifecycle Pattern

- `FileManager` tracks file states in explicit maps: fetching, saving, saved, errored.
- Upload/download APIs are injected, making file storage backend-agnostic at class boundary.
- Image element status updates are derived from file save outcomes.
- Unload-prevention logic is tied to in-flight saves rather than persisted element flags.

## Encryption-First Sharing Pattern

- Collaboration payloads are encrypted before websocket emission in `Portal._broadcastSocketData`.
- Shared scenes/files persisted in Firebase use ciphertext + IV, with decrypt on read.
- Share-link import/export (`data/index.ts`) uses compress/decompress flows with encryption keys.
- Room keys are generated client-side and embedded in collaboration URLs.

## Firebase Persistence Pattern

- Firebase app/storage/firestore clients are lazily initialized and cached in module scope.
- Scene writes use Firestore transactions with reconciliation against previously stored scene.
- File blobs are uploaded to Firebase Storage with cache-control metadata.
- Scene version cache (per socket) avoids redundant writes and unnecessary unload blocking.

## Progressive Initialization Pattern

- App entry registers PWA service worker (`virtual:pwa-register`) and boots React root.
- Language initialization is asynchronous (`InitializeApp`) and blocks editor render until ready.
- Collaboration-only dependencies (`socket.io-client`, random username helper) are lazy-loaded.
- Editor mount is followed by async scene restoration, then a later `editor:initialize` lifecycle event after loading completes.

## Build/Packaging Pattern

- Package build scripts produce separate `dist/dev` and `dist/prod` outputs (esbuild-based).
- Build-time env is injected as `import.meta.env` for dev/prod modes.
- `@excalidraw/excalidraw` exports are environment-aware (development/production entry mapping).
- Docker build compiles app assets, then serves static output via Nginx runtime image.

## Details

For detailed architecture → see [architecture.md](../technical/architecture.md)  
For undocumented runtime contracts → see [undocumented-behaviors.md](../technical/undocumented-behaviors.md)  
For product requirements → see [PRD.md](../product/PRD.md)  
For domain glossary → see [domain-glossary.md](../product/domain-glossary.md)
