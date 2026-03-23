# Bug, Chore & Triage Templates Design

**Date:** 2026-03-22
**Issue:** #18 — Capture skill expansion with bug/chore/triage templates
**Approach:** Lean templates, rich brainstorm guide (Approach 3)

## Design Decisions

### Hierarchy Model

Bugs and chores sit differently relative to the PRD > PI > Epic > Feature > Story hierarchy:

- **Bugs are peers** — they don't belong inside the hierarchy. No parent epic or parent feature. They interact with the hierarchy solely through dependencies (blocks/blocked-by). A bug found while working on Feature #12 references it via `Blocks: #12`, not via a parent relationship.
- **Chores are flexible** — they can be standalone peers (like bugs) or parented to an epic/feature. A chore to "update React" is standalone. A chore to "clean up test fixtures from feature X" genuinely belongs to feature X. Parent fields are optional in frontmatter.
- **Triage is raw intake** — unsorted, unclassified. Gets promoted to a real type via `sdlc:define`.

### Severity vs Priority (Bugs Only)

Bugs have both `severity` and `priority` as distinct concepts:

| Severity | Meaning |
|----------|---------|
| `critical` | System unusable, data loss, security vulnerability |
| `high` | Major feature broken, no workaround |
| `medium` | Feature degraded, workaround exists |
| `low` | Cosmetic, edge case, minor annoyance |

Severity describes impact. Priority describes when to fix it. A cosmetic bug on the homepage could be low severity but high priority because it's customer-facing.

### Lean Templates, Rich Brainstorm Guide

Following the established project pattern: templates are rigid output formats (section names + one-line descriptions), brainstorm guides are where investigation depth lives. The bug brainstorm guide covers reproduction steps, regression detection, root cause analysis — all of which feed into a clean, lean issue body.

## Templates

Each template file starts with a preamble paragraph explaining what it is and how it's used, consistent with existing templates (story-template.md, feature-template.md, etc.).

### Bug Template (`templates/bug-template.md`)

Preamble: "The canonical output format for Bug drafts. Bugs are GitHub Issues with `type:bug` label. Bugs are peers to the hierarchy — they have no parent epic or feature, only dependency relationships."

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

Required sections:
- `## Description` — What's broken, when/how it manifests, any error messages or symptoms
- `## Reproduction Steps` — Steps to reproduce the bug (or "Not yet reproduced" with known context)
- `## Expected vs Actual Behavior` — What should happen vs what does happen
- `## Affected Areas` — Which parts of the system are impacted
- `## File Scope` — Known files involved (may be partial at capture, filled by define)
- `## Dependencies` — Bidirectional: `- Blocked by: #N` / `- Blocks: #N` (or `none`)

No Parent section. Bugs are always peers to the hierarchy. Severity lives in frontmatter, not as a body section — it's structured metadata, not prose.

### Chore Template (`templates/chore-template.md`)

Preamble: "The canonical output format for Chore drafts. Chores are GitHub Issues with `type:chore` label. Chores can be standalone (no parent) or parented to an epic/feature."

```yaml
---
type: chore
name: <chore name>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
parent-epic: <issue number>    # optional
parent-feature: <issue number> # optional
---
```

Required sections:
- `## Description` — What needs to be done and why
- `## Task` — The specific work to perform
- `## Acceptance Criteria` — Checklist of conditions for "done"
- `## File Scope` — Files to create/modify
- `## Dependencies` — Bidirectional: `- Blocked by: #N` / `- Blocks: #N` (or `none`)

Conditional sections:
- `## Parent` — Only if `parent-epic` or `parent-feature` is set. Format: `Epic: #N, Feature: #N`

Validation rules:
- If `parent-epic` or `parent-feature` is set, `## Parent` section MUST exist
- If neither parent is set, `## Parent` section MUST be omitted

### Triage Template (`templates/triage-template.md`)

Preamble: "The canonical output format for Triage captures. Triage issues are raw intake — unsorted and unclassified. They get promoted to a real type via `sdlc:define`."

**Label note:** The GitHub label for triage is `triage` (no `type:` prefix), unlike `type:bug` and `type:chore`. This matches the existing capture skill behavior. The frontmatter `type: triage` is for internal draft processing only.

```yaml
---
type: triage
name: <issue name>
areas: [<area labels>]
status: draft
---
```

Required sections:
- `## Description` — What was observed or reported
- `## Initial Analysis` — Codebase exploration findings (related files, recent git history, open issues)
- `## Possible Causes` — Hypotheses based on the analysis
- `## Suggested Next Steps` — What to do: define as feature? investigate as bug? handle as chore?
- `## Dependencies` — Bidirectional format (or `none`)

Lightest frontmatter — no priority, no severity, no parents. Unsorted intake. No File Scope section (premature for triage).

## Bug Brainstorm Guide (`skills/define/reference/bug-brainstorm.md`)

Internal checklist for the define skill, woven into natural conversation:

**Before you can draft, make sure you understand:**
- What exactly is broken? (symptoms, error messages, behavior)
- What's the expected behavior vs actual behavior?
- Can it be reproduced? What are the steps?
- Is this a regression or long-standing? (check git history)
- What's the severity? (critical/high/medium/low)
- Which areas of the system are affected?
- Are there related bugs or issues?
- What's the fix scope? (one file, multiple areas, architectural)
- Does this block or get blocked by any existing work?

**Context to load:**
- Search codebase for files related to the bug's symptoms
- Check recent git history for changes in affected areas
- Check open issues for related work: `gh issue list --label "type:bug"`
- Read `.claude/sdlc/prd/PRD.md` — check Architecture and Security Constraints if relevant

**Research agent:** For bugs with unclear scope — trace code paths, identify affected files, check test coverage, look for similar patterns.

**Template reference:** `${CLAUDE_PLUGIN_ROOT}/templates/bug-template.md`

**Note:** Bugs have no size/decomposition concept (unlike features with size:small/large). A bug is always a single unit of work.

## Chore Brainstorm Handling

Chores reuse `story-brainstorm.md` as specified in issue #18. However, the story guide's parent-related questions ("Which parent feature and epic does this belong to?") should be treated as conditional for chores — standalone chores skip these. Define's Phase 1 should note this when loading the story brainstorm guide for a chore.

## Pre-Flight Checks for New Types

Add to define's pre-flight checks table:

| Level | Prerequisites |
|-------|--------------|
| Bug | PRD exists. No parent requirements — bugs are peers. |
| Chore | PRD exists. If parented, parent epic/feature resolvable via `gh issue view`. |

## Deviations from Issue #18 ACs

These are deliberate design decisions from brainstorming, not oversights:

| AC Specification | Design Decision | Rationale |
|-----------------|-----------------|-----------|
| Bug template has `Parent` section | No Parent section | Bugs are peers to the hierarchy — brainstormed and confirmed. Dependencies cover all bug-to-hierarchy relationships. |
| `Expected Behavior` and `Actual Behavior` as separate sections | Merged into `Expected vs Actual Behavior` | These are always read together. One section with clear contrast is more useful than two tiny sections. |
| `Severity` as a body section | Severity in frontmatter only | Severity is structured metadata (enum value), not prose. Frontmatter is the right place, consistent with how `priority` is handled. |

The AC should be updated to reflect these decisions before implementation begins.

## Remaining ACs (Straightforward Implementation)

These ACs were not brainstormed — the user assessed them as straightforward:

- Capture type detection: prefix-based (`bug:`, `chore:`) with fallback to keyword inference and user confirmation
- Capture grounding statement: "CAPTURE, DON'T DESIGN" → "CAPTURE WITH CONTEXT"
- Capture hard gate: prohibit full brainstorming, allow clarifying questions and user review
- Light-ceremony behavior per type (bug/chore/triage) as specified in issue #18
- Define Phase 1: recognize `bug` and `chore` as types, load appropriate brainstorm guide, detect type from existing issue labels on reshape
- Label assignment: `type:bug`, `type:chore`, or `triage` on created issues

## File Scope Summary

**Create:**
- `templates/bug-template.md`
- `templates/chore-template.md`
- `templates/triage-template.md`
- `skills/define/reference/bug-brainstorm.md`

**Modify:**
- `skills/capture/SKILL.md` — type detection, new grounding, new hard gate, template loading, triage exploration
- `skills/define/SKILL.md` — Phase 1 type recognition for bug/chore, brainstorm guide loading, label detection on reshape
