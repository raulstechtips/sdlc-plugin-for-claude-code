# Chore Template

The canonical output format for Chore drafts. Chores are GitHub Issues with `type:chore` label. Chores can be standalone (no parent) or parented to an epic/feature.

## Frontmatter

```yaml
---
type: chore
name: <chore name>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
parent-epic: <issue number>    # optional
parent-feature: <issue number> # optional
---
```

## Required Sections

- `## Description` — What needs to be done and why
- `## Task` — The specific work to perform
- `## Acceptance Criteria` — Checklist of conditions for "done"
- `## File Scope` — Files to create/modify
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)

## Conditional Sections

- `## Parent` — **Only if `parent-epic` or `parent-feature` is set**. **Omit entirely if standalone.**
  - Both set: `Epic: #<parent-epic>, Feature: #<parent-feature>`
  - Only epic set: `Epic: #<parent-epic>`
  - Only feature set: `Feature: #<parent-feature>`

## Validation Rules

- If `parent-epic` or `parent-feature` is set, `## Parent` section MUST exist
- If neither parent is set, `## Parent` section MUST be omitted
