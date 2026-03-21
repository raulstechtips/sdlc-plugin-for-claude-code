# Story Template

The canonical output format for Story drafts. Stories are GitHub Issues with `type:story` label. Stories are always leaf nodes — they belong to a feature and have no children.

## Frontmatter

```yaml
---
type: story
name: <story name>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
parent-epic: <issue number>
parent-feature: <issue number>
---
```

## Required Sections

- `## Description` — What single task to accomplish
- `## Acceptance Criteria` — Checklist of specific, testable conditions
- `## File Scope` — Lists of files to create and modify
- `## Technical Notes` — Implementation considerations, patterns to follow
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)
- `## Parent` — `Epic: #<parent-epic>, Feature: #<parent-feature>`
