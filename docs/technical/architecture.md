# Architecture

## Snapshot

- Snapshot date: **April 23, 2026**.
- Scope: architecture of the current repository implementation.
- Evidence policy: statements below are derived from repository source files only.

## High-level Architecture

### Runtime Structure

- The runtime has a product shell in `excalidraw-app` and an editor core in `packages/excalidraw`.
- The product shell renders `<Excalidraw />` and adds app-specific UX for collaboration, sharing, persistence, and integrations.
- The editor core (`packages/excalidraw/components/App.tsx`) owns canvas interaction, scene updates, actions, store/history integration, and rendering orchestration.
- Element model and scene container are implemented in `packages/element` (`Scene`, `Store`, delta utilities).
- External systems used by app-level flows include:
- WebSocket server URL from `import.meta.env.VITE_APP_WS_SERVER_URL`.
- Firebase config from `import.meta.env.VITE_APP_FIREBASE_CONFIG`.
- Share-link backend URLs from `import.meta.env.VITE_APP_BACKEND_V2_GET_URL` and `VITE_APP_BACKEND_V2_POST_URL`.
- AI backend URL from `import.meta.env.VITE_APP_AI_BACKEND`.

### High-level Mermaid Diagram

```mermaid
flowchart LR
  U[User Input: mouse keyboard touch]

  subgraph APP[excalidraw-app]
    APPROOT[App.tsx wrapper]
    COLLAB[collab/Collab.tsx + Portal.tsx]
    LOCAL[LocalData + localStorage + IndexedDB]
    SHARE[share/ShareDialog.tsx + data/index.ts]
    AIFLOW[components/AI.tsx]
  end

  subgraph CORE[@excalidraw/excalidraw]
    EXC[Excalidraw component]
    COREAPP[components/App.tsx]
    ACT[actionManager]
    RECON[packages/excalidraw/data/reconcile.ts]
    SCN[Scene from @excalidraw/element]
    STR[Store from @excalidraw/element]
    HIST[History]
    REND[Renderer + canvases]
  end

  subgraph ELEM[@excalidraw/element]
    E_TYPES[element types]
    E_RECON[delta + indices utilities]
    E_RENDER[renderElement]
  end

  subgraph EXT[External Services]
    WS[WS server]
    FB[(Firebase Firestore/Storage)]
    SHBACK[Share backend]
    AI[(AI backend)]
  end

  U --> APPROOT
  APPROOT --> EXC
  EXC --> COREAPP
  COREAPP --> ACT
  ACT --> COREAPP
  COREAPP --> SCN
  COREAPP --> STR
  STR --> HIST
  COREAPP --> REND
  COREAPP --> RECON
  REND --> E_RENDER
  SCN --> E_TYPES

  COREAPP -->|onChange callback| APPROOT
  APPROOT --> LOCAL
  APPROOT --> COLLAB
  APPROOT --> SHARE
  APPROOT --> AIFLOW

  COLLAB --> WS
  COLLAB --> FB
  SHARE --> SHBACK
  SHARE --> FB
  AIFLOW --> AI
```

### Layer Responsibilities

- `excalidraw-app`:
- Owns route/hash/bootstrap behavior for external scenes and collaboration links.
- Owns app-level dialogs, menu composition, share UX, collab trigger surfaces.
- Owns local persistence orchestration (`LocalData`) and collaboration adapter (`Collab`).
- `@excalidraw/excalidraw`:
- Owns editor state machine and action dispatch.
- Owns scene object lifecycle and store/history increments.
- Owns canvas rendering orchestration and editor event handling.
- `@excalidraw/element`:
- Owns scene element map/array, index normalization, mutation utilities.
- Owns delta model and store snapshot/change calculations.
- Owns low-level element rendering primitives used by renderer modules.

## Data Flow: how data moves through the system

### Flow A: Initial Scene Bootstrap

1. `excalidraw-app/App.tsx::initializeScene` reads URL search/hash and local browser data.
2. Local state is loaded with `importFromLocalStorage()`, then `restoreElements()` and `restoreAppState()` are applied.
3. If hash matches `#json=<id>,<key>`, `importFromBackend(id, key)` fetches and decrypts share payload.
4. If hash matches collaboration room (`#room=<id>,<key>`), app calls `collabAPI.startCollaboration(...)`.
5. The resolved scene is passed through `initialStatePromiseRef` into `<Excalidraw initialData={...} />`.
6. After initial scene, image files are resolved from:
7. local IndexedDB (`LocalData.fileStorage.getFiles`) for local sessions,
8. Firebase Storage (`loadFilesFromFirebase`) for external/shared scenes,
9. collab file flow (`collabAPI.fetchImageFilesFromFirebase`) for active collaboration.

### Flow B: Local Editing and Change Emission

1. Pointer/keyboard/touch events are handled inside `packages/excalidraw/components/App.tsx`.
2. User commands go through `ActionManager` (`handleKeyDown`, `executeAction`, panel actions).
3. Actions call `perform(...)` and return `ActionResult`.
4. `App.syncActionResult(...)` applies:
5. element updates via `scene.replaceAllElements(...)`,
6. app state updates via `this.setState(...)`,
7. file additions via `addMissingFiles(...)`.
8. `syncActionResult` schedules store action with `store.scheduleAction(actionResult.captureUpdate)`.
9. In `componentDidUpdate`, `store.commit(elementsMap, this.state)` computes and emits increments.
10. Also in `componentDidUpdate`, if `!isLoading`, editor triggers `props.onChange(elements, appState, files)`.
11. In `excalidraw-app`, this `onChange` callback drives:
12. `collabAPI.syncElements(elements)` when collaboration is active,
13. `LocalData.save(elements, appState, files, ...)` when local save is not paused.

### Flow C: Local Persistence and Cross-tab Sync

1. `LocalData.save(...)` debounces writes (`SAVE_TO_LOCAL_STORAGE_TIMEOUT`).
2. Scene/app state are persisted to localStorage (`STORAGE_KEYS.LOCAL_STORAGE_ELEMENTS`, `LOCAL_STORAGE_APP_STATE`).
3. Binary files are persisted to IndexedDB via `idb-keyval`.
4. Version keys (`VERSION_DATA_STATE`, `VERSION_FILES`) are updated by `tabSync.ts`.
5. On focus/visibility events, app checks `isBrowserStorageStateNewer(...)`.
6. If newer data exists:
7. scene/app state are restored with `excalidrawAPI.updateScene(...)`,
8. missing files are loaded from `LocalData.fileStorage.getFiles(...)`.

### Flow D: Collaboration (Realtime + Persistence)

1. `Collab.startCollaboration(...)`:
2. lazily imports `socket.io-client`,
3. resolves room id/key (existing hash or generated),
4. opens socket with `VITE_APP_WS_SERVER_URL`,
5. pauses local save with `LocalData.pauseSave("collaboration")`.
6. `Portal` encrypts outbound socket payloads in `_broadcastSocketData(...)` using `encryptData(...)`.
7. Inbound payloads are decrypted in `Collab.decryptPayload(...)`.
8. Scene updates from peers are reconciled via `_reconcileElements(...)` using `reconcileElements(...)`.
9. Remote-applied updates use `captureUpdate: CaptureUpdateAction.NEVER`.
10. File payloads are handled through `FileManager` + Firebase storage (`saveFilesToFirebase`, `loadFilesFromFirebase`).
11. Scene snapshots are persisted in Firestore with `saveToFirebase(...)` transactions.
12. `syncElements(...)` triggers both:
13. immediate delta broadcast (`broadcastElements`),
14. throttled full-scene broadcast and Firebase save (`queueBroadcastAllElements`, `queueSaveToFirebase`).

### Flow E: Share-link Export and Import

1. Export path:
2. `exportToBackend(elements, appState, files)` serializes scene with `serializeAsJSON(...)`.
3. payload is compressed+encrypted (`compressData(..., { encryptionKey })`).
4. encrypted payload is POSTed to `VITE_APP_BACKEND_V2_POST_URL`.
5. files are encoded/encrypted and uploaded to Firebase under `/files/shareLinks/<id>`.
6. resulting URL hash is `#json=<id>,<encryptionKey>`.
7. Import path:
8. `importFromBackend(id, key)` fetches from `VITE_APP_BACKEND_V2_GET_URL + id`.
9. payload is decompressed/decrypted and parsed into `ImportedDataState`.
10. legacy decoder path is kept as fallback for older buffer format.

## State Management: detailed description (appState, elements, actionManager)

### appState Ownership and Lifecycle

- Primary runtime app state is React component state in `packages/excalidraw/components/App.tsx`.
- Initial values are composed from `getDefaultAppState()` plus runtime props and viewport offsets.
- `AppState` type in `packages/excalidraw/types.ts` defines:
- interaction state (active tool, selection, editing, resizing, rotating),
- viewport state (scroll, zoom, dimensions, offsets),
- UI state (dialogs, menus, sidebar, theme, welcome, errors),
- collaboration state (collaborators map, follow mode),
- persistence/export settings (file handle, export flags, grid settings).
- `AppStateObserver` provides selector/predicate-based subscriptions and flushes on `componentDidUpdate`.

### Elements and Scene Ownership

- Elements are not stored in React state.
- Elements are stored in `Scene` (`packages/element/src/Scene.ts`) with:
- `elements` (including deleted),
- `nonDeletedElements`,
- `elementsMap` and `nonDeletedElementsMap`,
- cached selected-elements lookup.
- Scene updates call `triggerUpdate()`, regenerating `sceneNonce` and notifying listeners.
- In editor mount, `scene.onUpdate(this.triggerRender)` is registered.

### ActionManager in the Update Loop

- `ActionManager` is created in editor constructor with:
- updater callback (`syncActionResult`),
- appState getter,
- elements getter,
- app instance.
- `registerAll(actions)` loads the action registry.
- Undo/redo actions are added explicitly via `createUndoAction(history)` and `createRedoAction(history)`.
- `handleKeyDown(...)` filters actions by `keyTest`, `keyPriority`, and canvas action availability.
- `executeAction(...)` and panel callbacks call action `perform(...)`, then forward result to updater.

### `syncActionResult` Contract

- `syncActionResult(actionResult)` is the centralized integration point for action outputs.
- It always schedules capture policy using `store.scheduleAction(actionResult.captureUpdate)`.
- It conditionally applies:
- element replacement (`scene.replaceAllElements`),
- file ingestion and image cache scheduling,
- app state merge (`setState`).
- If nothing changed, it forces scene re-render via `scene.triggerUpdate()`.

### Store and History Integration

- `Store` lives in `@excalidraw/element` and tracks snapshot deltas.
- Capture modes are:
- `IMMEDIATELY` (durable/undoable),
- `EVENTUALLY` (ephemeral now, durable on later immediate capture),
- `NEVER` (ephemeral, not undoable).
- `updateScene(...)` API can schedule `store.scheduleMicroAction(...)` with explicit capture mode.
- `componentDidUpdate(...)` calls `store.commit(elementsMap, this.state)` on every render cycle.
- In `componentDidMount(...)`, editor subscribes:
- `store.onDurableIncrementEmitter -> history.record(increment.delta)`.
- `store.onStoreIncrementEmitter -> props.onIncrement` (only when prop exists).
- `History` maintains undo/redo stacks as `HistoryDelta` entries and re-applies via store micro actions.

### Observed AppState for Delta Efficiency

- `getObservedAppState(...)` in `packages/element/src/store.ts` projects full `AppState` to a smaller observed shape.
- Observed keys include: `name`, `viewBackgroundColor`, selection/group ids, linear selection editing info, crop id, lock ids.
- Store snapshot/delta logic uses observed app state for change calculations.

### App-level State on Top of Editor Core

- `excalidraw-app` uses a separate Jotai store (`appJotaiStore`) for app shell state:
- collaboration API atom,
- collaboration/offline flags,
- share dialog state,
- language atom,
- local storage quota flag.
- Editor package uses isolated Jotai scope (`packages/excalidraw/editor-jotai.ts`) for internal atoms.

## Rendering Pipeline: from React component to canvas

### 1) Render Preparation in `App.render()`

- `App.render()` calculates `selectedElements` from `Scene`.
- It retrieves `sceneNonce`.
- It calls `renderer.getRenderableElements(...)` with:
- viewport (`zoom`, `scrollX`, `scrollY`, `width`, `height`, offsets),
- editing state (`editingTextElement`, `newElementId`),
- `sceneNonce`.
- Result is `{ elementsMap, visibleElements }`.

### 2) Renderer Filtering and Memoization

- `Renderer.getRenderableElements(...)`:
- builds a renderable map excluding:
- currently edited text element,
- current `newElement` by id.
- computes visible elements via `isElementInViewport(...)`.
- memoizes result keyed by viewport/editing inputs and `sceneNonce`.

### 3) Canvas Layer Composition

- The editor renders three canvas layers in `App.render()`:
- `StaticCanvas` for main scene painting.
- `NewElementCanvas` when `appState.newElement` exists.
- `InteractiveCanvas` for selection UI, handles, remote pointers, scrollbars, and interaction overlays.

### 4) Static Scene Rendering

- `StaticCanvas` mounts a dedicated canvas node and keeps its pixel size synced to app dimensions and scale.
- `StaticCanvas` calls `renderStaticScene(...)`.
- `renderStaticScene(...)`:
- bootstraps context and applies zoom transform,
- optionally draws grid based on `renderGrid`,
- iterates visible elements and paints via `renderElement(...)`,
- applies frame clip rules and link icon overlays,
- uses `renderConfig` inputs such as `imageCache`, `theme`, erasure/embeddable metadata.

### 5) New Element Preview Rendering

- `NewElementCanvas` calls `renderNewElementScene(...)`.
- The renderer:
- bootstraps canvas and applies zoom,
- renders only `newElement` (if not selection and not invisibly small),
- applies optional frame clipping,
- draws with `renderElement(...)`.

### 6) Interactive Scene Rendering

- `InteractiveCanvas` computes collaborator pointer/selection maps from `appState.collaborators`.
- It prepares renderer params and starts `AnimationController` with key `animateInteractiveScene`.
- Animation loop calls `renderInteractiveScene(...)`.
- `renderInteractiveScene(...)` delegates to `_renderInteractiveScene(...)`, then calls callback with:
- `scrollBars`,
- `atLeastOneVisibleElement`,
- `elementsMap`.
- Editor callback updates `currentScrollBars`, computes `scrolledOutside`, and schedules image refresh.

### 7) Render-trigger Sources

- `Scene.triggerUpdate()` invalidates scene nonce and notifies callbacks.
- React state changes also trigger render.
- On mount, editor registers `scene.onUpdate(this.triggerRender)`.
- Render throttling switches are wired via `isRenderThrottlingEnabled()` and `window.EXCALIDRAW_THROTTLE_RENDER`.

## Package Dependencies: relationships between packages

### Workspace Packages

- `@excalidraw/common`
- `@excalidraw/math`
- `@excalidraw/element`
- `@excalidraw/utils`
- `@excalidraw/excalidraw`

### Dependency Graph from `package.json` Files

```mermaid
flowchart TD
  C[@excalidraw/common]
  M[@excalidraw/math]
  E[@excalidraw/element]
  U[@excalidraw/utils]
  X[@excalidraw/excalidraw]

  M --> C
  E --> C
  E --> M
  X --> C
  X --> E
  X --> M
```

### Notes About Linking and Build Boundaries

- Root `tsconfig.json` maps `@excalidraw/*` aliases directly to local package sources (`packages/*/src` and `packages/excalidraw/*`).
- `vitest.config.mts` mirrors these aliases for test runtime.
- `excalidraw-app` imports `@excalidraw/excalidraw` and other `@excalidraw/*` modules through these local alias mappings.
- `scripts/buildPackage.js`:
- aliases `@excalidraw/utils` to `packages/utils/src`,
- marks `@excalidraw/common`, `@excalidraw/element`, `@excalidraw/math` as externals.
- `packages/excalidraw/package.json` exports typed subpaths for `./common/*`, `./element/*`, `./math/*`, and `./utils/*`.

### Package-level Responsibility Split

- `@excalidraw/common`: shared constants, utilities, event helpers, environment helpers.
- `@excalidraw/math`: geometry and vector utilities.
- `@excalidraw/element`: element model, mutation, scene container, render primitives, selection/group/frame helpers.
- `@excalidraw/utils`: utility helpers consumed via aliasing/build paths and typed exports.
- `@excalidraw/excalidraw`: editor composition, actions, renderer orchestration, public React API.

## Source Verification

- App shell orchestration: `excalidraw-app/App.tsx`, `excalidraw-app/index.tsx`.
- Collaboration and transport: `excalidraw-app/collab/Collab.tsx`, `excalidraw-app/collab/Portal.tsx`.
- Share/import/export flows: `excalidraw-app/data/index.ts`, `excalidraw-app/share/ShareDialog.tsx`.
- Firebase persistence: `excalidraw-app/data/firebase.ts`.
- Local persistence and cross-tab sync: `excalidraw-app/data/LocalData.ts`, `excalidraw-app/data/localStorage.ts`, `excalidraw-app/data/tabSync.ts`.
- Editor core state loop: `packages/excalidraw/components/App.tsx`, `packages/excalidraw/actions/manager.tsx`, `packages/excalidraw/history.ts`, `packages/excalidraw/types.ts`, `packages/excalidraw/appState.ts`.
- Scene/store implementation: `packages/element/src/Scene.ts`, `packages/element/src/store.ts`.
- Render pipeline: `packages/excalidraw/scene/Renderer.ts`, `packages/excalidraw/components/canvases/StaticCanvas.tsx`, `packages/excalidraw/components/canvases/NewElementCanvas.tsx`, `packages/excalidraw/components/canvases/InteractiveCanvas.tsx`, `packages/excalidraw/renderer/staticScene.ts`, `packages/excalidraw/renderer/renderNewElementScene.ts`, `packages/excalidraw/renderer/interactiveScene.ts`.
- Package wiring and dependencies: `packages/common/package.json`, `packages/math/package.json`, `packages/element/package.json`, `packages/utils/package.json`, `packages/excalidraw/package.json`, `tsconfig.json`, `vitest.config.mts`, `scripts/buildPackage.js`.
