# Bug Template

The canonical output format for Bug drafts. Bugs are GitHub Issues with `type:bug` label. Bugs are peers to the hierarchy — they have no parent epic or feature, only dependency relationships.

## Frontmatter

```yaml
---
type: bug
name: <bug name>
severity: <critical|high|medium|low>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
---
```

## Required Sections

- `## Description` — What's broken, when/how it manifests, any error messages or symptoms
- `## Reproduction Steps` — Steps to reproduce the bug (or "Not yet reproduced" with known context)
- `## Expected vs Actual Behavior` — What should happen vs what does happen
- `## Affected Areas` — Which parts of the system are impacted
- `## File Scope` — Known files involved (may be partial at capture, filled by define)
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)
