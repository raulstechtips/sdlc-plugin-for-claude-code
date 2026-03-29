# Bug Template

The canonical output format for Bug drafts. Bugs are GitHub Issues with `type:bug` label. Bugs can be standalone (no parent) or optionally parented to a PI, epic, or feature.

## Frontmatter

```yaml
---
type: bug
name: <bug name>
severity: <critical|high|medium|low>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
parent-pi: <issue number>      # optional
parent-epic: <issue number>    # optional
parent-feature: <issue number> # optional
---
```

## Required Sections

- `## Description` — What's broken, when/how it manifests, any error messages or symptoms
- `## Reproduction Steps` — Steps to reproduce the bug (or "Not yet reproduced" with known context)
- `## Expected vs Actual Behavior` — What should happen vs what does happen
- `## Affected Areas` — Which parts of the system are impacted
- `## File Scope` — Known files involved (may be partial at capture, filled by define)
- `## Technical Notes` — Implementation considerations, root cause analysis, patterns to follow
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)

## Conditional Sections

- `## Parent` — **Only if `parent-pi`, `parent-epic`, or `parent-feature` is set**. **Omit entirely if standalone.**
  - Show each set parent in hierarchy order: `PI: #<parent-pi>`, `Epic: #<parent-epic>`, `Feature: #<parent-feature>`
  - Only include lines for parents that are set

## Validation Rules

- If any parent field is set, `## Parent` section MUST exist
- If no parent fields are set, `## Parent` section MUST be omitted
