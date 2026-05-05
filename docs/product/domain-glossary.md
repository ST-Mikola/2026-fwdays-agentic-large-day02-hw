# Domain Glossary

## Snapshot

- Snapshot date: **April 23, 2026**.
- Scope: project-specific terminology used in this Excalidraw codebase.
- Evidence policy: definitions are grounded in repository source code only.

## Terms

### Element

- Definition in this project:
- A drawable unit managed by the editor scene and passed through actions/rendering as element arrays/maps.
- In practice, editor logic operates on both deleted and non-deleted elements, depending on flow.
- Where used (key files):
- `packages/element/src/types.ts` (element type family and element aliases).
- `packages/element/src/Scene.ts` (scene-owned element collections and lookups).
- `packages/excalidraw/components/App.tsx` (reads/writes scene elements in editor lifecycle).
- Not to confuse with:
- General UI/DOM “element” (HTML node in the browser DOM).
- In this project, “Element” means Excalidraw scene data objects, not DOM nodes.

### ExcalidrawElement

- Definition in this project:
- Core union type for persisted scene objects (`rectangle`, `diamond`, `ellipse`, `text`, linear/arrow, freedraw, image, frame, magicframe, iframe, embeddable).
- Explicitly documented as JSON-serializable and shareable between peers, with no peer-local computed state.
- Where used (key files):
- `packages/element/src/types.ts` (type definition and related wrappers).
- `packages/excalidraw/types.ts` (`BinaryFiles` keyed by element id, API/event typings).
- `excalidraw-app/data/index.ts` (share/collab payload typing).
- Not to confuse with:
- A React component type or runtime canvas primitive.
- Here it is a serializable domain model object for scene state.

### Scene

- Definition in this project:
- A state container class that owns current elements, non-deleted subsets, element maps, selected-elements cache, and a render invalidation nonce.
- Provides mutation/query APIs such as `replaceAllElements()`, `getNonDeletedElements()`, `getElementsMapIncludingDeleted()`, and `getSelectedElements(...)`.
- Where used (key files):
- `packages/element/src/Scene.ts` (class implementation).
- `packages/excalidraw/components/App.tsx` (creates `new Scene()`, subscribes via `scene.onUpdate(...)`, updates via `scene.replaceAllElements(...)`).
- Not to confuse with:
- A generic “screen” or route/page in frontend apps.
- In this project, Scene is the authoritative in-memory container of drawing elements.

### AppState

- Definition in this project:
- Editor runtime/UI state interface (tooling, selection, editing modes, viewport, menus/dialogs, theme, collaboration UI state, snapping/cropping/search, etc.).
- Default values are produced by `getDefaultAppState()` and merged with runtime props on app initialization.
- Where used (key files):
- `packages/excalidraw/types.ts` (full `AppState` interface).
- `packages/excalidraw/appState.ts` (`getDefaultAppState()` and storage/export config).
- `packages/excalidraw/components/App.tsx` (main React state owner).
- Not to confuse with:
- Global application state for the whole monorepo.
- Here it is editor-instance state (canvas session/UI), not all product-shell state.

### Tool

- Definition in this project:
- Editor drawing/interaction mode represented by `ToolType` (e.g. `selection`, `lasso`, `rectangle`, `arrow`, `text`, `image`, `eraser`, `hand`, `frame`, `laser`, etc.).
- Active tool is modeled through `ActiveTool` (regular tool or `custom`) and embedded into `AppState.activeTool` with lock/temporary-switch metadata.
- Where used (key files):
- `packages/excalidraw/types.ts` (`ToolType`, `ActiveTool`, `AppState.activeTool`).
- `packages/excalidraw/appState.ts` (default active tool configuration).
- Not to confuse with:
- External dev tooling (build/test tools).
- In this project, “Tool” means user-facing editor mode on canvas.

### Action

- Definition in this project:
- Typed command object with `name`, optional keyboard/predicate metadata, and `perform(...)` function.
- `perform(...)` returns `ActionResult` (elements/appState/files updates + capture policy) or `false`.
- Where used (key files):
- `packages/excalidraw/actions/types.ts` (`Action`, `ActionResult`, `ActionName`, `ActionSource`).
- `packages/excalidraw/components/App.tsx` (`syncActionResult(...)` applies action outputs).
- Not to confuse with:
- Redux action objects or REST actions.
- Here it is an executable editor command contract.

### ActionManager

- Definition in this project:
- Dispatcher/registry for editor actions; resolves keyboard matches, executes actions from UI/API, and routes results to the updater callback.
- Owns action registration lifecycle (`registerAction`, `registerAll`) and keyboard handling (`handleKeyDown`).
- Where used (key files):
- `packages/excalidraw/actions/manager.tsx` (class implementation).
- `packages/excalidraw/components/App.tsx` (instantiation and action registration).
- Not to confuse with:
- A generic command bus across the whole app shell.
- In this project, it manages editor action execution inside Excalidraw core.

### Collaboration

- Definition in this project:
- Real-time co-editing layer exposed via `CollabAPI` and implemented by `Collab` + `Portal`.
- Uses websocket events/subtypes for init/update/pointer/idle/visible-bounds sync and encrypts collaboration payloads before transport.
- Where used (key files):
- `excalidraw-app/collab/Collab.tsx` (`CollabAPI`, session lifecycle, `startCollaboration`, `syncElements`, `stopCollaboration`).
- `excalidraw-app/collab/Portal.tsx` (socket transport boundary).
- `excalidraw-app/app_constants.ts` (`WS_SUBTYPES`, websocket event constants).
- `excalidraw-app/data/index.ts` (collaboration link parsing and socket payload types).
- Not to confuse with:
- Async share-link export/import.
- In this project, collaboration is a live room-based sync session, separate from static link sharing.

### Library

- Definition in this project:
- Reusable shape/template collection represented by `LibraryItem` / `LibraryItems`, managed by the editor library subsystem.
- Supports persisted storage through adapters and update flows via `updateLibrary(...)` and `useHandleLibrary(...)`.
- Where used (key files):
- `packages/excalidraw/types.ts` (`LibraryItem`, `LibraryItems`, `LibraryItemsSource`).
- `packages/excalidraw/data/library.ts` (library state/update/persistence orchestration + hook).
- `excalidraw-app/App.tsx` (integration via `useHandleLibrary(...)` with IndexedDB adapter/migration adapter).
- Not to confuse with:
- npm package libraries in the monorepo.
- Here “Library” means end-user drawing asset collection inside the editor.

### CaptureUpdateAction

- Definition in this project:
- Capture policy enum-like constant for store/history behavior: `IMMEDIATELY`, `EVENTUALLY`, `NEVER`.
- Controls whether updates are recorded in local undo/redo immediately, later, or never.
- Where used (key files):
- `packages/element/src/store.ts` (definition + scheduling/commit semantics).
- `packages/excalidraw/components/App.tsx` (`store.scheduleAction(actionResult.captureUpdate)`).
- `excalidraw-app/collab/Collab.tsx` (remote/collab updates use non-history capture paths).
- Not to confuse with:
- UI event propagation or React batching settings.
- In this project, it is a history-capture policy for scene/app-state updates.

### Store

- Definition in this project:
- Change-capture engine that snapshots observed editor state, calculates deltas, and emits durable/ephemeral increments.
- Serves as the bridge between scene/app-state mutations and history/increment subscribers.
- Where used (key files):
- `packages/element/src/store.ts` (`Store`, `StoreChange`, `StoreDelta`, snapshot/increment logic).
- `packages/excalidraw/components/App.tsx` (store creation, `commit(...)`, subscriptions).
- `packages/excalidraw/history.ts` (history records durable deltas from store).
- Not to confuse with:
- Jotai atom store or global state library by itself.
- In this project, Store is specifically the delta/increment pipeline for editor state capture.

### ExcalidrawImperativeAPI

- Definition in this project:
- Public imperative interface exposed by editor instance (`updateScene`, `applyDeltas`, `getSceneElements...`, `getAppState`, `getFiles`, `updateLibrary`, event subscriptions, etc.).
- Used by the app shell and integrations to control/query editor behavior outside declarative props alone.
- Where used (key files):
- `packages/excalidraw/types.ts` (interface contract).
- `packages/excalidraw/components/App.tsx` (`createExcalidrawAPI()` implementation).
- `excalidraw-app/App.tsx` (product-shell orchestration through API instance).
- Not to confuse with:
- REST API or backend service API.
- In this project, it is an in-browser object API of one editor instance.

### Bound Element

- Definition in this project:
- An element that participates in explicit binding metadata instead of just being visually near another element.
- Common examples in this codebase are text bound to a container and arrows bound to bindable elements through `boundElements`, `startBinding`, or `endBinding`.
- Where used (key files):
- `packages/element/src/types.ts` (binding-related fields on element types).
- `packages/element/src/binding.ts` (binding creation/update/removal logic).
- `packages/element/src/typeChecks.ts` (`isBoundToContainer` and related checks).
- `packages/excalidraw/components/App.tsx` (editor flows that preserve and update bindings).
- Not to confuse with:
- Grouped elements or frame membership.
- In this project, a bound element is linked through binding metadata with specific editor behavior.

### Linear Element

- Definition in this project:
- A specific element family represented by `ExcalidrawLinearElement`, covering `line` and `arrow`.
- Its geometry is stored as `points`, and it supports endpoint binding, arrowheads, and editor-specific manipulation via `LinearElementEditor`.
- Where used (key files):
- `packages/element/src/types.ts` (`ExcalidrawLinearElement`, `ExcalidrawLineElement`, `ExcalidrawArrowElement`).
- `packages/element/src/linearElementEditor.ts` (editing model and point operations).
- `packages/excalidraw/components/App.tsx` (selection/editing state and action flows).
- `packages/excalidraw/renderer/interactiveScene.ts` (interactive rendering/editor affordances).
- Not to confuse with:
- Any generic straight line in graphics code.
- In this project, “Linear Element” is a first-class scene type with point-based editable geometry.
