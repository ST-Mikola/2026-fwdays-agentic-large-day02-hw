# Product Requirements Document (PRD) for Excalidraw

**Owner:** Product and Engineering team  
**Last updated:** April 28, 2026  
**Version:** 1.1  
**Status:** Working draft aligned to the current repository snapshot

## 1. Document Scope

- This PRD describes the product requirements for the Excalidraw user-facing experience implemented in this repository.
- Primary scope: the `excalidraw-app` web application.
- Secondary scope: first-class adjacent product capabilities exposed through this repository, including share/export flows, collaboration, AI-assisted workflows, and the embeddable `@excalidraw/excalidraw` library.
- This document is not a backend implementation spec and does not define the standalone collaboration/share/AI services in detail.

## 2. Product Overview

### 2.1. Overview

Excalidraw is a browser-native, open-source virtual whiteboard for sketches, diagrams, and lightweight visual communication. Its core promise is low-friction idea capture with a distinctive hand-drawn feel, while still supporting collaboration, sharing, and developer integration.

### 2.2. Problem

People frequently need to sketch ideas, flows, architecture, and UI concepts quickly. Many existing tools are too heavy, too slow to start, overly structured, or require installation and onboarding before users can express an idea.

### 2.3. Solution

Excalidraw provides:

- instant drawing with minimal setup,
- reliable local persistence so users do not lose work,
- secure live collaboration and encrypted sharing flows,
- export and integration paths for broader reuse,
- a reusable React library for embedding the editor into other products.

## 3. Target Users and Jobs

### 3.1. Solo Creator

- **Role:** Developer, designer, product manager, educator, content creator.
- **Primary job:** Capture and refine an idea quickly.
- **Needs:** Fast startup, simple drawing tools, local recovery, export in useful formats.

### 3.2. Collaborative Team

- **Role:** Small product, design, or engineering team.
- **Primary job:** Discuss and edit the same scene live.
- **Needs:** Easy room sharing, visible collaborator presence, low-latency updates, safe session lifecycle.

### 3.3. Share-Link Consumer

- **Role:** Teammate, stakeholder, or external viewer opening a shared scene.
- **Primary job:** Access an exported scene through a link without setup friction.
- **Needs:** Simple access, correct scene restoration, secure link-based sharing.

### 3.4. Developer Integrator

- **Role:** Frontend engineer embedding Excalidraw into another web product.
- **Primary job:** Reuse the editor as a supported React component.
- **Needs:** Stable package surface, documentation, examples, and integration flexibility.

## 4. Product Goals

### 4.1. Business Goals

- Grow usage and community adoption of the OSS product.
- Maintain a healthy ecosystem around the embeddable `@excalidraw/excalidraw` package.
- Provide a clear upgrade and handoff path into Excalidraw+ without degrading core OSS workflows.

### 4.2. Product Goals

- Minimize time-to-first-drawing.
- Preserve user trust with strong “don’t lose work” behavior.
- Make both live collaboration and encrypted scene sharing first-class workflows.
- Keep the interface simple for casual users while preserving discoverability for power users.
- Support adjacent workflows such as export, integration, and AI-assisted experimentation.

## 5. Functional Requirements

### 5.1. Instant Sketching

- **Priority:** Must-have
- **Description:** A user should be able to open the app and begin drawing immediately without registration or setup.
- **User Story:** As a user, I want to start drawing right away so I can capture an idea before context is lost.
- **Acceptance Criteria:**
  - On first launch, the editor opens to an interactive canvas with primary drawing controls available.
  - Core entry points such as the welcome UI, main menu, and toolbar do not block drawing.
  - The default experience does not require account creation or mandatory onboarding.

### 5.2. Local Persistence and Recovery

- **Priority:** Must-have
- **Description:** The app should persist scene state locally and restore recoverable work across reloads and browser sessions.
- **User Story:** As a user, I want my work to come back after a refresh or accidental interruption so I do not lose progress.
- **Acceptance Criteria:**
  - Scene elements and app state are saved to browser storage automatically.
  - Binary assets required by the scene are persisted separately and restored when available.
  - On reload or when newer local data is detected, the app can restore the previous session state.
  - The app protects against common accidental-loss scenarios during active editing or in-flight save flows.

### 5.3. Shareable Scene Links

- **Priority:** Should-have
- **Description:** Users should be able to generate a reusable encrypted link for a scene snapshot.
- **User Story:** As a user, I want to share a scene with other people through a link so they can open the same content easily.
- **Acceptance Criteria:**
  - The user can export a scene to a shareable link from the app UI.
  - Shared scene payloads are encrypted before being persisted to external storage.
  - Opening a valid scene link restores the corresponding scene in the app.
  - Errors in share-link generation or restoration are surfaced clearly to the user.

### 5.4. Real-time Collaboration

- **Priority:** Must-have
- **Description:** Multiple users should be able to work on the same scene in real time through a room-based collaboration flow.
- **User Story:** As a team member, I want to invite collaborators into a live session so we can edit the same board together.
- **Acceptance Criteria:**
  - The user can start or join a collaboration room through a shareable room link.
  - Participants can see remote changes reflected in near real time.
  - Participants can see collaborator presence signals such as cursors and related session state.
  - Collaboration payloads are encrypted end-to-end.
  - The user can stop the session explicitly and the app handles the exit flow safely.
  - The app provides user feedback when collaboration is affected by offline state or service issues.

### 5.5. Export and Interoperability

- **Priority:** Should-have
- **Description:** Users should be able to take their work out of the editor and reuse it in other contexts.
- **User Story:** As a user, I want to export or hand off my work so I can use it in presentations, documents, or adjacent Excalidraw workflows.
- **Acceptance Criteria:**
  - The app supports local export of scenes in common formats used by Excalidraw workflows.
  - The app supports an explicit handoff path to Excalidraw+ where available.
  - Export-related actions are discoverable from the app UI and command-oriented flows.

### 5.6. Embeddable Library for Developers

- **Priority:** Should-have
- **Description:** The repository should continue to expose a reusable React package for embedding Excalidraw into external products.
- **User Story:** As a frontend developer, I want a supported component package so I can integrate Excalidraw into my own application.
- **Acceptance Criteria:**
  - `@excalidraw/excalidraw` is published as a reusable package from this repository.
  - The package has documented setup guidance and integration examples.
  - The library can render the editor in a host application without requiring the full product shell.

### 5.7. AI-Assisted Workflows

- **Priority:** Could-have
- **Status:** Experimental
- **Description:** The product may support AI-assisted workflows that extend the canvas experience but do not replace the core editor.
- **User Story:** As a user, I want optional AI helpers so I can transform diagrams or prompts into useful outputs faster.
- **Acceptance Criteria:**
  - Diagram-to-code can send selected visual context to an AI backend and render a returned result.
  - Text-to-diagram can submit prompt-driven requests and stream responses back into the UI.
  - AI features degrade gracefully when the AI backend is unavailable or rate-limited.
  - AI workflows remain optional and do not block the primary drawing experience.

## 6. Feature Priority and Current Status

| Capability | Priority | Current Status |
|---|---|---|
| Instant sketching | Must-have | Implemented |
| Local persistence and recovery | Must-have | Implemented |
| Real-time collaboration | Must-have | Implemented |
| Shareable encrypted scene links | Should-have | Implemented |
| Export and Excalidraw+ handoff | Should-have | Implemented |
| Embeddable React library | Should-have | Implemented |
| AI-assisted workflows | Could-have | Experimental / implemented behind service dependency |

## 7. Non-Functional Requirements

- **Performance:** The product should feel immediate to start and remain responsive during normal editing. Time-to-first-drawing should target under 1 second in a healthy local environment.
- **Reliability:** The product should optimize for “don’t lose work” behavior through local persistence, safe restore paths, and guarded session transitions.
- **Security and Privacy:** Collaboration and link-sharing payloads must be encrypted end-to-end, and encryption keys must remain client-side.
- **Availability:** Collaboration, sharing, and AI flows depend on external services and should fail gracefully when those services are unavailable.
- **Scalability:** Collaboration should support small-team synchronous use without major latency or scene corruption.
- **Compatibility:** The app assumes modern browser support for canvas, storage, and related APIs required by the editor.
- **Global Readiness:** The product should support internationalized UI and explicit language switching.

## 8. Dependencies and Constraints

### 8.1. External Dependencies

- Collaboration requires a WebSocket-based collaboration service.
- Shared-scene and file persistence depend on Firebase-backed storage flows.
- Shareable link export/import depends on external share-link backend endpoints.
- AI-assisted workflows depend on a separate AI backend service.

### 8.2. Product Constraints

- The repository is frontend-first; not all deployed backend services live in this codebase.
- Collaboration is not the same flow as static scene-link sharing and must remain a distinct user journey.
- Full offline-first sync beyond local browser persistence is out of scope.
- Collaboration behavior may be restricted in embedding scenarios such as iframes.

## 9. Success Metrics

The repository documentation does not currently define trusted numeric baselines or production analytics targets for all flows. Until those are formalized, this PRD tracks the metrics categories below:

- Activation: time-to-first-drawing and successful editor starts.
- Retention and continuity: restore usage, save reliability, and work-loss incidents.
- Collaboration adoption: rooms created, sessions with more than one active participant, collaboration completion rate.
- Sharing adoption: number of shareable scene links generated and successfully reopened.
- Ecosystem adoption: package downloads, example usage, and external integrations of `@excalidraw/excalidraw`.
- Commercial handoff: usage of Excalidraw+ export or upgrade entry points.
- Community health: GitHub stars, contributors, and community participation signals.

Exact targets should be added once product analytics ownership and baseline measurements are agreed.

## 10. Out of Scope

- Implementing or documenting the full standalone collaboration backend in this repository.
- Enterprise-only access features such as SSO, advanced role-based access control, or admin policy management for the OSS app.
- Guaranteed fully offline collaboration or multi-device sync beyond local browser persistence.
- Treating experimental AI flows as mandatory core editor functionality.

## 11. Release View

This repository snapshot supports the following release-oriented interpretation:

- **Core editor:** implemented and production-relevant.
- **Local persistence and restore:** implemented and core to the UX promise.
- **Live collaboration:** implemented, but operationally dependent on external services.
- **Shareable encrypted links:** implemented, but operationally dependent on external services.
- **Excalidraw+ handoff:** implemented as an adjacent export flow.
- **Embeddable library:** implemented as a maintained public package.
- **AI-assisted workflows:** present in the product surface, but should be treated as experimental because they depend on external service availability and evolving UX.

## 12. Related Documents

- Product and UX context: `docs/memory/productContext.md`
- Repository scope and goals: `docs/memory/projectbrief.md`
- Technical architecture: `docs/technical/architecture.md`
- Domain terminology: `docs/product/domain-glossary.md`
