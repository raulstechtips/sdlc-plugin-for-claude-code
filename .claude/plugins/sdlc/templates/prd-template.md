# PRD Template

The canonical output format for PRD drafts. Used by define (drafting), create (validation), and draft-reviewer (completeness checks).

## Frontmatter

```yaml
---
name: <project name>
version: <N.N>
created: <YYYY-MM-DD>
status: draft
---
```

## Required Sections

All sections below are required. Each must have non-empty content (no placeholders).

- `## Overview` — What the project is and why it exists
- `## Tech Stack` — Languages, frameworks, tools
- `## Architecture` — High-level design, key decisions
- `## Data Models` — Artifact types, label taxonomy, data structures
- `## API Contracts` — External interfaces (CLI tools, APIs)
- `## Security Constraints` — Auth, data sensitivity, access control
- `## Roadmap` — High-level phases of work
- `## Acceptance Criteria` — Checkboxes defining "done" for the product
- `## Out of Scope` — Explicit exclusions
- `## Label Taxonomy` — Category table with labels and purposes
- `## Decision Log` — Table: Date, Decision, Reason, Affects
