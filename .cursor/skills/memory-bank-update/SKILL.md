---
name: memory-bank-update
description: Workflow for updating the Memory Bank with verified project changes.
---

# Skill: Memory Bank Update

## Invocation

- Invoked by `.cursor/rules/memory-bank.mdc`
- Can also be used on an explicit request to update or sync the Memory Bank

## Inputs

- What changed (from git diff, conversation, or user description)

## Steps

1. Inspect the relevant diff or user request to identify what changed
2. Read relevant Memory Bank files in `docs/memory/`
3. For each changed area, update the matching file:
    - New feature → `progress.md` + `activeContext.md`
    - Architecture change → `systemPatterns.md` + `decisionLog.md`
    - Dependency change → `techContext.md`
    - Scope change → `projectbrief.md` + `productContext.md`
4. Verify updated content against actual source code
5. Ensure each file stays under 200 lines

## Outputs

- List of updated Memory Bank files
- Summary of what changed and why

## Safety

- Do NOT remove manually curated content without asking
- Do NOT add speculative information — only verified facts
- Do NOT exceed 200 lines per file — summarize if needed
- Verify ALL technical claims against actual code
