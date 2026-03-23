# PI Template

The canonical output format for PI (Program Increment) drafts. PI is a GitHub Issue with `type:pi` label.

## Frontmatter

```yaml
---
type: pi
name: PI-<N>
theme: <one-line theme>
priority: high
areas: [<area labels>]
status: draft
---
```

## Required Sections

- `## Goals` — What this PI aims to achieve overall
- `## Timeline` — Start and target dates
- `## Epics` — One subsection per epic (see format below)
- `## Dependencies` — Cross-epic dependencies only
- `## Worktree Strategy` — Which epics can be parallelized

## Timeline Format

```markdown
## Timeline
- Started: YYYY-MM-DD
- Target: YYYY-MM-DD
```

## Epic Subsection Format

Each epic within `## Epics` uses this format. The `(#TBD)` placeholder is replaced with the real issue number when the epic is created.

```markdown
### Epic: <name> (#TBD)
**Goal:** <what success looks like — one or two sentences>
**Priority:** critical | high | medium | low
**Scope seeds:**
- <bullet point — rough chunk of work>
- <bullet point — these become feature candidates later>
- <bullet point — not commitments, just direction>
```

Scope seeds are starting points for epic brainstorming, not commitments.
