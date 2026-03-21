# PI Template

The canonical output format for PI (Program Increment) drafts. Lighter than previous versions — epics are goals with scope seeds, not detailed feature lists.

## Frontmatter

```yaml
---
name: PI-<N>
theme: <one-line theme>
started: <YYYY-MM-DD>
target: <YYYY-MM-DD>
status: active
---
```

## Required Sections

- `## Goals` — What this PI aims to achieve overall
- `## Epics` — One subsection per epic (see format below)
- `## Dependencies` — Cross-epic dependencies only
- `## Worktree Strategy` — Which epics can be parallelized

## Epic Subsection Format

Each epic within `## Epics` uses this format:

```markdown
### Epic: <name>
**Goal:** <what success looks like — one or two sentences>
**Priority:** critical | high | medium | low
**Scope seeds:**
- <bullet point — rough chunk of work>
- <bullet point — these become feature candidates later>
- <bullet point — not commitments, just direction>
```

Scope seeds are starting points for epic brainstorming, not commitments. No issue numbers or `#TBD` placeholders at this stage.
