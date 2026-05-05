# Project Brief

## What This Project Is

- This is an Excalidraw monorepo (`name: excalidraw-monorepo`) managed with Yarn workspaces.
- The main end-user product is the `excalidraw-app` web application.
- The repository also contains reusable npm packages: `@excalidraw/excalidraw`, `@excalidraw/common`, `@excalidraw/element`, `@excalidraw/math`, `@excalidraw/utils`.
- The repository includes integration examples such as `examples/with-nextjs` and `examples/with-script-in-browser`.
- The main app is built with React, TypeScript, and Vite, while the monorepo also includes package build scripts and framework-specific examples.

## Primary Goal

- Deliver a browser-based collaborative whiteboard and diagramming experience with Excalidraw's hand-drawn visual style.
- Maintain an embeddable library (`@excalidraw/excalidraw`) for integration into external React applications.
- Support live collaboration workflows through the `excalidraw-app/collab` layer.
- Support scene and file persistence through Firebase-backed flows.
- Protect collaboration and shared-scene data with encryption in supported sync/storage paths.

## Target Users

- End users of the Excalidraw web app (drawing, sharing scenes, collaboration).
- Developers who need a ready-to-use Excalidraw React component in their products.
- The core team maintaining the editor engine and publishing related packages from one repo.

## Scope

- The repository contains the frontend app, reusable libraries, tests, and integration examples.
- It also includes Firebase rules/config (`firebase-project/*`) and containerization config (`Dockerfile`, `docker-compose.yml`).
- The architecture is frontend-first; collaboration client logic is present here, while the standalone real-time backend is not implemented as a separate service in this tree.

## Core Value

- Single source of truth for the Excalidraw product app.
- Single source of truth for public integration packages.
- Single source of truth for local end-to-end development across app and packages through workspace links.
- Enables parallel evolution of app UX and library API without version drift.

## Constraints And Non-Goals

- This file describes the repository and product at a high level, not the full internal architecture.
- The repo contains examples and build tooling, but it is not only a demo project for the public package.
- Real-time collaboration depends on external services and configuration, so this repository alone is not the whole deployed system.

## Success Criteria

- Stable `excalidraw-app` startup in dev and production modes.
- Reliable build and publish-ready outputs for `packages/*`.
- Working integration flows in `examples/*`.
- Working collaboration and persistence flows (socket + Firebase).

## Details

For detailed architecture → see [architecture.md](../technical/architecture.md)  
For undocumented runtime contracts → see [undocumented-behaviors.md](../technical/undocumented-behaviors.md)  
For product requirements → see [PRD.md](../product/PRD.md)  
For domain glossary → see [domain-glossary.md](../product/domain-glossary.md)
