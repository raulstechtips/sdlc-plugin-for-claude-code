# Chore Template

The canonical output format for Chore drafts. Chores are GitHub Issues with `type:chore` label. Chores can be standalone (no parent) or optionally parented to a PI, epic, or feature.

## Frontmatter

```yaml
---
type: chore
name: <chore name>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
parent-pi: <issue number>      # optional
parent-epic: <issue number>    # optional
parent-feature: <issue number> # optional
---
```

## Required Sections

- `## Description` — What needs to be done and why
- `## Task` — The specific work to perform
- `## Acceptance Criteria` — Checklist of conditions for "done"
- `## File Scope` — Files to create/modify
- `## Technical Notes` — Implementation considerations, patterns to follow
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)

## Conditional Sections

- `## Parent` — **Only if `parent-pi`, `parent-epic`, or `parent-feature` is set**. **Omit entirely if standalone.**
  - Show each set parent in hierarchy order: `PI: #<parent-pi>`, `Epic: #<parent-epic>`, `Feature: #<parent-feature>`
  - Only include lines for parents that are set

## Validation Rules

- If any parent field is set, `## Parent` section MUST exist
- If no parent fields are set, `## Parent` section MUST be omitted
