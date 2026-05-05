# Undocumented Behaviors

## Snapshot

- Snapshot date: **April 23, 2026**.
- Scope: implementation behaviors that are observable in source code but are not explicitly documented in technical/product docs.
- Discovery method: targeted review of `HACK`, `FIXME`, `TODO` comments and surrounding runtime logic.

## Undocumented Behavior #1

- **File**: `packages/excalidraw/components/App.tsx` (module-level flags around lines 587-589, hack note around 689-693, touch flow around 3588-3632)
- **What happens**: double-tap/double-click logic is implemented as an implicit cross-event state machine using module-level mutable flags (`didTapTwice`, `firstTapPosition`, `tappedTwiceTimer`) and mixed browser/manual handling.
- **Where documented**: only inline comments (`TODO this is a hack...`), not in `docs/technical/architecture.md`.
- **Risk**: refactors that "simplify" click handling can break mobile text insertion and touch interaction edge cases.

## Undocumented Behavior #2

- **File**: `packages/excalidraw/components/App.tsx` (lines 5735-5750)
- **What happens**: text submit path depends on explicit `flushSync(setState)` before finalize-like flows; selection must be updated first to keep subsequent behavior correct.
- **Where documented**: only inline TODO (`move this into finalize... or handle all state updates in one place`).
- **Risk**: changing call order may desync selection state, finalize behavior, and undo/redo capture.

## Undocumented Behavior #3

- **File**: `packages/excalidraw/components/App.tsx` (lines 6325-6345, 6360-6364)
- **What happens**: browser double-click handling is conditionally gated by manual click-history heuristics (`shouldHandleBrowserCanvasDoubleClick`) to avoid conflicting native vs touch semantics.
- **Where documented**: only inline TODO (`remove this once we consolidate double-click logic`).
- **Risk**: threshold or condition changes can create phantom double-clicks or missed intent on touch devices.

## Undocumented Behavior #4

- **File**: `packages/excalidraw/components/App.tsx` (lines 7126-7133)
- **What happens**: explicit HACK disables transform handles for linear elements on mobile (and for 2-point cases) inside cursor/transform detection path.
- **Where documented**: only inline `HACK` comment.
- **Risk**: UI/interaction optimizations around handles may regress mobile editing behavior.

## Undocumented Behavior #5

- **File**: `packages/excalidraw/components/App.tsx` (line 8758 and surrounding text-on-pointer-down flow)
- **What happens**: text insertion container resolution has a known unresolved branch (`FIXME`) where hit element/container rewrites pointer insertion context.
- **Where documented**: nowhere except inline `FIXME`.
- **Risk**: future changes can silently break bound-text placement in container elements.

## Undocumented Behavior #6

- **File**: `packages/element/src/store.ts` (lines 101-112, 391-405)
- **What happens**: capture semantics are an implicit priority state machine (`IMMEDIATELY > NEVER > EVENTUALLY`) and `scheduleCapture()` is noted as error-prone due to widespread call sites.
- **Where documented**: partially described at high level in architecture docs, but precedence rules and scheduling invariants are not explicitly documented.
- **Risk**: AI-generated "cleanup" of capture scheduling can corrupt history, increment emission, and synchronization behavior.

## Undocumented Behavior #7

- **File**: `packages/excalidraw/data/restore.ts` (lines 405-411)
- **What happens**: restoring with `deleteInvisibleElements` mutates text elements into deleted state and bumps version, while inline TODO notes this breaks delta-based sync/versioning assumptions.
- **Where documented**: only inline TODO.
- **Risk**: restore-path refactors can introduce collaboration divergence or non-replayable deltas.

## Undocumented Behavior #8

- **File**: `packages/excalidraw/actions/actionFinalize.tsx` (lines 142-147, 232-236, 346-347)
- **What happens**: finalize marks invisibly-small elements deleted and still captures broad state with `CaptureUpdateAction.IMMEDIATELY`; comments acknowledge inconsistency risk if narrowed.
- **Where documented**: only inline TODO notes in finalize action.
- **Risk**: "optimize capture scope" changes may cause empty or inconsistent undo/redo histories.

## Undocumented Behavior #9

- **File**: `packages/element/src/sizeHelpers.ts` (lines 27-29)
- **What happens**: known systemic inconsistency: invisibly-small elements are not cleaned uniformly across actions/store/export/broadcast/persistence.
- **Where documented**: only inline TODO; no central cleanup contract documented.
- **Risk**: localized fixes can move bugs between pipelines rather than resolve them.

## Undocumented Behavior #10

- **File**: `packages/excalidraw/data/library.ts` (lines 248-258)
- **What happens**: library teardown intentionally does not reset `libraryItemsAtom` because editor Jotai store is not scoped per instance yet.
- **Where documented**: only inline TODO in `destroy()`.
- **Risk**: multi-instance/remount scenarios may leak or reuse stale library UI state.

## Undocumented Behavior #11

- **File**: `packages/excalidraw/wysiwyg/textWysiwyg.tsx` (lines 964-969)
- **What happens**: WYSIWYG theme refresh relies on `onChangeEmitter` side channel because Store does not emit theme updates yet.
- **Where documented**: only inline `FIXME`.
- **Risk**: changing event emission paths may leave editor text UI with stale styling/theme.

## Undocumented Behavior #12

- **File**: `excalidraw-app/collab/Collab.tsx` (lines 499-503, 512-565)
- **What happens**: collaboration startup has multiple initialization paths (socket connect-error fallback, timeout fallback, socket `INIT` path), and scene promise typing is acknowledged as overloaded/abused.
- **Where documented**: only inline TODO; no explicit race/precedence contract in docs.
- **Risk**: simplifying startup control flow may introduce race conditions, duplicate initialization, or dropped initial scene.

## Undocumented Behavior #13

- **File**: `packages/excalidraw/components/App.tsx` (lines 1549-1585)
- **What happens**: embeddables are lazily initialized by visibility; once an embed becomes visible, its id is added to `initializedEmbeds`, and later renders keep it renderable even when it is no longer currently visible.
- **Where documented**: observable in render logic, but not explicitly described in technical docs.
- **Risk**: viewport/render optimizations can accidentally change embed lifecycle, reinitialization behavior, or offscreen persistence expectations.

## Undocumented Behavior #14

- **File**: `packages/excalidraw/components/App.tsx` (line 2083 and surrounding render path)
- **What happens**: the render path mutates instance state via `this.visibleElements = visibleElements` after renderer output is computed, so other subsystems can consume the latest visible subset outside React state.
- **Where documented**: not documented in architecture/product docs.
- **Risk**: assuming render is pure, or moving visibility bookkeeping elsewhere, can break consumers that depend on `app.visibleElements` being refreshed during render.

## Undocumented Behavior #15

- **File**: `packages/excalidraw/components/App.tsx` (lines 3358-3364)
- **What happens**: `editor:initialize` and `onInitialize` are emitted once from `componentDidUpdate`, only after `isLoading` becomes `false`, not directly on mount.
- **Where documented**: only inferable from lifecycle code; not explicitly called out in current docs.
- **Risk**: lifecycle refactors can shift API readiness timing and break integrations that rely on initialization happening after async scene restoration.

## Notes

- These behaviors are not necessarily bugs; they are fragile runtime contracts currently encoded in implementation details.
- Before refactoring these areas, add characterization tests and document expected event/order semantics first.

## Source Verification

- `packages/excalidraw/components/App.tsx`
- `packages/element/src/store.ts`
- `packages/excalidraw/data/restore.ts`
- `packages/excalidraw/actions/actionFinalize.tsx`
- `packages/element/src/sizeHelpers.ts`
- `packages/excalidraw/data/library.ts`
- `packages/excalidraw/wysiwyg/textWysiwyg.tsx`
- `excalidraw-app/collab/Collab.tsx`
