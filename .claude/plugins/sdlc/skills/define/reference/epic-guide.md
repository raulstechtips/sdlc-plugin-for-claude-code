# Epic Reference Guide

## Scope Assessment Criteria

| Signal | LIGHT | STANDARD | DEEP |
|--------|-------|----------|------|
| Area labels | 1 area | 2 areas | 3+ areas |
| Feature count | 1-2 features | 2-3 features | 5+ features |
| Pattern novelty | Purely follows existing patterns | Extends existing patterns with some new ground | Introduces new architectural patterns |
| Cross-epic dependencies | None | Intra-epic only | Has cross-epic dependencies |
| Estimated story count | 1-4 stories | 5-7 stories | 8+ stories |
| Questions needed | 2-3 | 4-5 | 6-7 |

**Quick rules:**
- DEEP if ANY: spans 3+ area labels, introduces new architectural patterns, has cross-epic dependencies, 5+ features
- STANDARD if ANY: 2-3 features, follows existing patterns with some new ground, has intra-epic dependencies
- LIGHT if ALL: 1-2 features, purely follows existing patterns, no cross-epic dependencies

## Context to Read

1. **PI Plan:** `.claude/sdlc/pi/PI.md` — find this epic's entry for its planned features and dependencies.
2. **PRD:** `.claude/sdlc/prd/PRD.md` — architecture, data models, security constraints, and tech stack context.

## Question Templates

### Questions (adapt count to depth)

1. **Overview:** "What does this epic deliver and why does it matter? (the value proposition)"
2. **Success criteria:** "How do we know it's done? (measurable checkboxes, not vague goals)"
3. **Features:** "What are the features? (confirm or adjust the PI's feature list)"
4. **Non-goals:** "What's explicitly out of scope for this epic?"
5. **Priority:** "What priority: critical, high, medium, or low?"
6. **Area labels:** "Which areas does this touch? (check the PRD's Label Taxonomy section at `.claude/sdlc/prd/PRD.md` for this project's valid areas)"
7. **Dependencies:** "Does it depend on or block other epics or features?"

### LIGHT: Focus on questions 1-3, confirm the rest from PI context.
### STANDARD: Ask questions 1-5, infer 6-7 from context if possible.
### DEEP: Ask all 7, plus follow-ups on architecture and dependencies.

## Feature Level: Optional Grouping

After determining the stories/features, check the count:

- If the epic has **fewer than ~8 stories total**, ask:

> "This epic has [N] stories. Want to group them into features, or keep them flat under the epic?"

- **Flat:** Stories are direct children of the epic. No feature issues. Simpler for small epics.
- **Features:** Group related stories under feature sub-headings. Better for larger or multi-area epics.
- If **8+ stories:** Recommend features for organization, but let the user decide.

## Draft Body Template

```markdown
---
type: epic
name: <epic name>
priority: <critical|high|medium|low>
areas: [<area1>, <area2>]
status: draft
---

## Overview
[What this epic delivers and why — 2-3 paragraphs covering the value proposition, the user/system impact, and how it fits into the broader PI goals]

## Success Criteria
- [ ] [measurable criterion 1]
- [ ] [measurable criterion 2]
- [ ] [measurable criterion 3]

## Features
- [ ] <Feature name> (#TBD) — [1-line description]
- [ ] <Feature name> (#TBD) — [1-line description]

## Non-goals
- [explicit exclusion 1]
- [explicit exclusion 2]

## Dependencies
- Blocked by: [#N, #M or "none"]
- Blocks: [#N, #M or "none"]
```

## Pre-Flight Checklist

- PRD exists at `.claude/sdlc/prd/PRD.md`
- PI exists at `.claude/sdlc/pi/PI.md`
- This epic is listed in the PI plan

## Discovery by Depth

### LIGHT (1-2 features, purely follows existing patterns, no cross-epic deps)
Summarize what you understand from the PI plan for this epic. Focus on questions 1-3 from templates above (overview, success criteria, features). Confirm the rest from PI context.

### STANDARD (2-3 features, extends patterns, intra-epic dependencies)
Ask questions 1-5 from templates above, one at a time. Infer area labels and dependencies from PI and PRD context if possible.

### DEEP (5+ features, new patterns, cross-epic deps, 3+ areas)
Ask all 7 questions from templates above plus follow-ups on architecture implications and cross-epic coordination.

For epics with fewer than ~8 stories total, ask: "This epic has [N] stories. Want to group them into features, or keep them flat under the epic?" If 8+ stories, recommend features for organization.

## Approaches by Depth

### LIGHT
Present one approach for the epic structure. Confirm with user.

### STANDARD
Present two options (e.g., different feature groupings, or flat vs features). Recommend one.
If the epic triggers the feature grouping decision (flat vs features), present it here.

### DEEP
Present three approaches with trade-off table (e.g., different decomposition strategies, different sequencing of features).

## Review Context

The draft reviewer should check this draft against:
- `.claude/sdlc/prd/PRD.md` — architecture alignment, label taxonomy consistency
- `.claude/sdlc/pi/PI.md` — epic is listed in PI, features match PI's feature list for this epic
- Internal consistency (success criteria match overview, features cover the scope described)
