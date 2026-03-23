# Triage Template

The canonical output format for Triage captures. Triage issues are raw intake — unsorted and unclassified. They get promoted to a real type via `sdlc:define`.

**Label note:** The GitHub label for triage is `triage` (no `type:` prefix), unlike `type:bug` and `type:chore`. This matches the existing capture skill behavior.

## Frontmatter

```yaml
---
type: triage
name: <issue name>
areas: [<area labels>]
status: draft
---
```

## Required Sections

- `## Description` — What was observed or reported
- `## Initial Analysis` — Codebase exploration findings (related files, recent git history, open issues)
- `## Possible Causes` — Hypotheses based on the analysis
- `## Suggested Next Steps` — What to do: define as feature? investigate as bug? handle as chore?
- `## Dependencies` — Bidirectional format (or `none`)
