# Feature Template

The canonical output format for Feature drafts. Features are GitHub Issues with `type:feature` label.

A feature can be directly implementable (`size:small`) or decomposable into stories (`size:large`).

## Frontmatter

```yaml
---
type: feature
name: <feature name>
priority: <critical|high|medium|low>
size: <small|large>
areas: [<area labels>]
status: draft
parent-epic: <issue number>
---
```

## Required Sections

- `## Description` — What this feature does (not how)
- `## Acceptance Criteria` — Checklist of measurable conditions for "done"
- `## Non-goals` — Explicit exclusions
- `## Technical Notes` — Implementation considerations, patterns, architectural decisions
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)
- `## Parent` — `Epic: #<parent-epic>`

## Conditional Sections

- `## File Scope` — **Only if `size: small`**. Lists of files to create and modify. **Omit entirely if `size: large`.**
- `## Stories` — **Only if `size: large`**. Checklist of stories with `(#TBD)` placeholders. **Omit entirely if `size: small`.**

## Section Order

**`size:small`:** Description → AC → Non-goals → File Scope → Technical Notes → Dependencies → Parent

**`size:large`:** Description → AC → Non-goals → Stories → Technical Notes → Dependencies → Parent

## Validation Rules

- `size:small` features MUST NOT have a `## Stories` section, MUST have a `## File Scope` section
- `size:large` features MUST have a `## Stories` section with at least one item, MUST NOT have a `## File Scope` section
