# Epic Template

The canonical output format for Epic drafts. Epics are GitHub Issues with `type:epic` label.

## Frontmatter

```yaml
---
type: epic
name: <epic name>
priority: <critical|high|medium|low>
areas: [<area labels>]
parent-pi: <issue number>
status: draft
---
```

## Required Sections

- `## Overview` — Goal/outcome, value proposition
- `## Success Criteria` — Checklist of measurable conditions for "done"
- `## Features` — Checklist of features with `(#TBD)` placeholders for issue numbers
- `## Non-goals` — Explicit exclusions
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)
- `## Parent` — `PI: #<N>` (links to parent PI issue)
