# Product Context

## Product UX Intent

- Deliver a low-friction, browser-native whiteboard for quick idea capture and diagramming.
- Support both solo work and real-time collaboration from the same canvas surface.
- Make sharing fast: collaborative room links and read-only shareable scene links are both first-class.
- Keep user trust high through privacy messaging, encrypted sharing/collab flows, and robust error handling.
- Provide an upgrade path to Excalidraw+ without blocking core OSS workflows.

## Primary User Groups

- Solo creators who need instant sketching, diagramming, and export.
- Small teams collaborating live on a shared scene.
- Users sharing snapshots or reusable links with broader audiences.
- Power users using keyboard-driven discovery (Command Palette) and advanced workflows.

## UX Goals

- Fast time-to-first-drawing via welcome hints and immediate access to load/help/collaboration actions.
- Frictionless collaboration with visible start/join entry points, rapid link sharing, and clear start/stop session lifecycle.
- Reliable “don’t lose work” behavior via debounced local persistence, file persistence, and unload protection during in-flight saves.
- Clear operational feedback through offline collaboration warnings, storage quota alerts, and collaboration error indicators/dialogs.
- Discoverability for casual and expert users through main menu defaults, command palette actions, and top-right utility controls.
- Global readiness through language detection and explicit language switching.

## Core UX Surfaces

- Canvas/editor surface: powered by `<Excalidraw>` with app-level wrappers.
- Welcome screen: guided entry points (load/help/live collaboration/sign up).
- Main menu: load, save/export, collaboration trigger, help, search, preferences, theme, language.
- Top-right utility area: collaboration trigger, promo banner, collab error indicator.
- Share dialog: collaboration session controls plus link-export flow.
- Sidebar promos: comments and presentation upsell panels.
- Footer: encryption indicator (for non-signed users) and optional debug controls.
- Command palette: collaboration/share/export/community/navigation shortcuts.

## Key User Scenarios

### 1) Start Drawing Immediately

- User opens app and lands on the editor with welcome hints and menu shortcuts.
- Editor is configured for immediate interaction (`autoFocus`, keyboard/global handling).
- If the app detects a previous local state, it can restore content and settings.

### 2) Resume Existing Work

- On load or tab refocus, app syncs with local browser storage versions.
- Elements/app state are restored from `localStorage`; image files from IndexedDB.
- Missing image files are rehydrated and stale statuses are updated.

### 3) Share a Read-Only Scene Link

- User opens Share dialog and selects link export.
- App exports encrypted/compressed payload to backend and returns share URL.
- Latest link is shown in `ShareableLinkDialog`; errors surface through `ErrorDialog`.

### 4) Start Live Collaboration

- User triggers collaboration from welcome screen, menu, top-right trigger, or command palette.
- App creates room link (or joins existing one), initializes socket transport, and syncs scene.
- Room dialog allows setting display name, copying link, native share, QR code sharing, and stopping session.

### 5) Collaborate Reliably In Real Time

- Remote updates are reconciled deterministically and applied as non-undoable remote updates.
- Pointer/location/idle/viewport events are transmitted and reflected in UI collaborator state.
- App warns when offline while in collaboration mode.

### 6) Exit Collaboration Safely

- Stop session flow persists current room data before disconnect.
- User gets explicit confirmation path when preserving remote state.
- Local save resumes and collaboration-specific state is reset after disconnect.

### 7) Export To Excalidraw+

- Export surface is available in export UI, overwrite-confirm action, and command palette.
- Scene + files are encrypted and uploaded, then user is redirected to Excalidraw+ import URL.
- Error states are handled and shown through app error channels.

### 8) Use AI-Assisted Workflows

- Diagram-to-code sends a frame image plus extracted text to an AI backend and renders returned HTML in an iframe-like result.
- Text-to-diagram streams model responses and persists chat history in IndexedDB.

### 9) Recover From App-Level Failures

- Top-level error boundary captures crashes, logs to Sentry, and shows recovery options.
- User can clear local storage and reload.
- UI provides path to open prefilled GitHub issue with captured event context.

## UX Guardrails And Constraints

- Collaboration is disabled when running inside iframe embedding.
- Self-embedding is explicitly guarded to prevent recursive embedding loops.
- Prevent-unload behavior can be toggled via env flag for specific environments.
- Privacy/crypto messaging is surfaced in sharing and footer UI (non-plus flow).
- PWA installation is opportunistic and gated by browser `beforeinstallprompt` heuristics.

## Product Signals In UX

- Frequent analytics events track load/share/export/collab interactions.
- Community entry points are embedded (GitHub, Discord, X, YouTube).
- Excalidraw+ upsell appears in menu, welcome screen, top banner, sidebar promos, and command palette.

## Details

For detailed architecture → see [architecture.md](../technical/architecture.md)  
For undocumented runtime contracts → see [undocumented-behaviors.md](../technical/undocumented-behaviors.md)  
For product requirements → see [PRD.md](../product/PRD.md)  
For domain glossary → see [domain-glossary.md](../product/domain-glossary.md)
