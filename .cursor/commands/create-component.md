Create a new component that matches existing Excalidraw repository patterns.

Inputs:
- `componentName`: desired component name
- Optional: target area or usage context

Workflow:
1. Normalize `componentName` to PascalCase for UI component files.
2. Decide whether the component belongs in `packages/excalidraw/components/` or `excalidraw-app/components/` based on the request and existing ownership boundaries.
3. Inspect nearby components in the target area and follow their local export style, props style, test placement, and styling conventions instead of forcing generic boilerplate.
4. If the component needs styles, create a sibling `{{ComponentName}}.scss` file and import it from the component. Prefer existing SCSS tokens and CSS variables over new raw values.
5. If the component introduces meaningful behavior, add or update tests using the nearest local pattern.
6. Do not add Storybook files by default; only add documentation when the public API or workflow change clearly needs it.
7. Summarize the files created or updated and mention any follow-up integration still needed.

Constraints:
- Prefer extending an existing component if that keeps the UI more consistent than introducing a new one.
- Keep helper/support files in local naming style such as `common.ts`, `types.ts`, `utils.ts`, or `index.tsx` where appropriate.
