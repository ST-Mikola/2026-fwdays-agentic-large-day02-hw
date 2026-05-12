Add a new translation key across Excalidraw locale files.

Inputs:
- `keyPath`: dot-separated locale path, for example `labels.myAction`
- `englishText`: English source text for `packages/excalidraw/locales/en.json`

Workflow:
1. Read `packages/excalidraw/locales/README.md` and respect the Crowdin-managed locale workflow.
2. Update `packages/excalidraw/locales/en.json` first. Add the new key at `keyPath`, creating missing nested objects only if needed.
3. Mirror the same key path into every other locale JSON file in `packages/excalidraw/locales/` with `""` as the value.
4. Do not edit `packages/excalidraw/locales/percentages.json`; it is generated.
5. Preserve the existing nesting and key order so other locales stay structurally aligned with `en.json`.
6. Report which locale files changed and call out that empty-string values still need translation.

Constraints:
- Do not rewrite unrelated translated strings.
- Keep the change limited to the new key unless the user explicitly asks for broader locale maintenance.
