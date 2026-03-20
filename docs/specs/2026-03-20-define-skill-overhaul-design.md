# Define Skill Overhaul — Design Spec

## Goal

Restructure `sdlc:define` to mirror the superpowers brainstorming skill's disciplined, native Claude Code patterns while preserving the SDLC plugin's depth-based discovery system (LIGHT/STANDARD/DEEP). The result is a skill that tracks progress via tasks, presents one phase at a time, runs automated draft review via a subagent, and enforces a human approval gate before transitioning to `sdlc:create`.

## Architecture

Three components:

1. **`SKILL.md`** (~150 lines) — Concise orchestrator with hardcoded 9-task checklist, hard gates, anti-rationalization table, and universal process framework. Mirrors brainstorming's structure. Owns all native Claude Code tooling (TaskCreate, TaskUpdate, Agent dispatch). Points to reference guides for depth-specific behavior.

2. **5 enriched reference guides** (`prd-guide.md`, `pi-guide.md`, `epic-guide.md`, `feature-guide.md`, `story-guide.md`) — Each guide is upgraded from read-only reference material to a self-contained playbook with depth-specific branching, pre-flight checklists, approach guidance, and review context. The guide is the single source of truth for *how* each task behaves at each depth level for that artifact type. Guides use zero native tooling — they are pure instruction content.

3. **`draft-reviewer` agent** (`agents/draft-reviewer/AGENT.md`) — Custom plugin agent dispatched by SKILL.md in Task 7. Reviews the draft file against upstream artifacts (PRD, PI, parent issues) appropriate to the level. Checks completeness, upstream consistency, internal consistency, dependency validity, scope, and YAGNI. Max 3 iterations before escalating to human.

### Separation of Concerns

| Component | Role | Native Tooling |
|-----------|------|----------------|
| SKILL.md | Conductor — creates tasks, manages transitions, enforces gates | TaskCreate, TaskUpdate, Agent dispatch |
| Reference guides | Sheet music — tells the conductor what to play at each phase | None (pure instruction content) |
| Draft reviewer agent | Quality gate — autonomous reviewer dispatched by conductor | Read, Bash, Grep, Glob (to read artifacts and check GitHub issues) |

## Plugin Structure

```
.claude/plugins/sdlc/
  plugin.json
  agents/
    draft-reviewer/
      AGENT.md
  skills/
    define/
      SKILL.md
      reference/
        prd-guide.md
        pi-guide.md
        epic-guide.md
        feature-guide.md
        story-guide.md
    init/
    capture/
    create/
    update/
    status/
    reconcile/
    retro/
```

## SKILL.md Design

### Frontmatter

```yaml
---
name: define
description: Use when defining new SDLC artifacts (PRD, PI, epic, feature, story) or reshaping existing ones through collaborative brainstorming that produces a reviewable local draft.
allowed-tools: Read, Edit, Write, Bash, Grep, Glob, Agent
argument-hint: "[level] [identifier]"
---
```

### Announcement

"I'm using the sdlc:define skill to define/reshape an SDLC artifact."

### Hard Gate

No draft without all tasks completed in order. No implementation, no creative shortcuts, no skipping tasks because the artifact "seems simple."

### Anti-Rationalization Table

~8 rows covering the key temptations:

| Thought | Reality |
|---------|---------|
| "This is simple, skip to the draft" | All tasks are mandatory. Simple artifacts go through LIGHT depth, not skipped tasks. |
| "I already know what to build" | Discovery catches assumptions. Even if you're right, the user confirms. |
| "The reference guide isn't needed" | The guide has scope criteria, question templates, and draft template. Always load it. |
| "Let me skip the approaches phase" | LIGHT gets a single approach confirmation. It takes 10 seconds. Don't skip it. |
| "The draft is obvious, no review needed" | The draft reviewer catches issues you miss after a long conversation. Always run it. |
| "I'll present multiple phases at once" | One task at a time. Complete each before moving to the next. |
| "I'll assess scope from intuition" | Scope criteria are objective and concrete in the reference guide. Use those signals, not vibes. |
| "The reviewer will catch everything" | The reviewer catches structural issues. Creative decisions are the user's job in Task 8. |

### Task Checklist

"You MUST create a task for each of these items and complete them in order:"

1. **Load context** — Parse level from `$ARGUMENTS`, load `reference/<level>-guide.md`, read upstream artifacts per guide's context table, detect new vs reshape
2. **Assess scope** — Evaluate depth (LIGHT/STANDARD/DEEP) using guide's objective criteria matrix, announce result with specific signals referenced
3. **Run discovery** — Ask questions one at a time per guide's depth-specific discovery instructions. Wait for each answer before asking the next. Dispatch research Agent when guide says to (STANDARD/DEEP).
4. **Re-evaluate scope** — Post-discovery gate: re-assess depth using same criteria. Depth can only go UP (max one upgrade). If upgraded, re-run discovery at new depth (don't re-ask answered questions).
5. **Present approaches** — LIGHT: 1 approach, confirm. STANDARD: 2 options with trade-offs, recommend one. DEEP: 3 with trade-off table. Guide may have level-specific approach considerations.
6. **Write draft** — Use guide's draft template. Write to `.claude/sdlc/drafts/<level>-<name>.md`. Reshapes include `## Changes` section. Frontmatter includes type, name, priority, areas, status, parent references.
7. **Draft review loop** — Dispatch `draft-reviewer` agent with: draft file path, level, upstream artifact paths from guide's review context section. Fix issues agent finds, re-dispatch (max 3 iterations). If still failing, surface issues to user.
8. **User reviews draft** — Present full draft to user. "Want to change anything?" Loop until explicit approval. Do NOT interpret silence or ambiguity as approval.
9. **Announce next step** — For new: "Draft saved to `<path>`. Run `/sdlc:create <level>` when ready to push it live." For reshape: "Draft saved with changes documented. Run `/sdlc:update <level> <number>` to apply the changes." Do NOT auto-invoke create or update.

### Universal Framework (kept from current SKILL.md)

- **Dependency format:** Canonical `- Blocked by: #N` / `- Blocks: #N` format
- **New vs reshape detection:** Issue number in args = reshape. "Reshape/rethink/revise/update" = reshape. Existing draft = ask user.
- **Reshape draft Changes section:** Required for all reshapes, consumed by `sdlc:update`
- **Integration:** define produces draft → create pushes to GitHub/git (new) or update applies edits (reshape)

## Reference Guide Enrichment

Each of the 5 existing guides gets these additions (existing content stays as-is):

### A. Pre-Flight Checklist

What must exist before this level can be defined:

| Level | Pre-Flight Requirements |
|-------|------------------------|
| PRD | Check if `.claude/sdlc/prd/PRD.md` exists (greenfield vs brownfield vs reshape) |
| PI | PRD exists. Check for previous retros in `.claude/sdlc/retros/`. Check for active PI. |
| Epic | PRD exists. PI exists. |
| Feature | PRD exists. PI exists. Parent epic issue resolvable via `gh issue view`. |
| Story | PRD exists. Parent feature and parent epic issues resolvable via `gh issue view`. |

### B. Depth-Specific Discovery Branching

Explicit instructions per depth level for how many questions to ask and which ones:

```markdown
## Discovery by Depth

### LIGHT
[Summarize from upstream context. Ask confirming question. Max N follow-ups.]
[Focus on questions: X, Y from templates above.]

### STANDARD
[Ask questions X-Y one at a time. Dispatch research Agent if needed.]

### DEEP
[Dispatch research Agent first. Then ask all questions plus follow-ups.]
```

Question numbers and counts are level-specific (already differ per guide).

### C. Depth-Specific Approach Guidance

```markdown
## Approaches by Depth

### LIGHT
Present one approach, confirm with user.

### STANDARD
Present two options with trade-offs. Recommend one.
[Level-specific consideration if applicable, e.g., epic's feature grouping decision.]

### DEEP
Present three approaches with trade-off table.
```

### D. Review Context

What the draft-reviewer agent should validate against for this level:

| Level | Review Against |
|-------|---------------|
| PRD | Codebase scan findings (brownfield), user's stated requirements |
| PI | `.claude/sdlc/prd/PRD.md` (roadmap alignment) |
| Epic | `.claude/sdlc/prd/PRD.md` + `.claude/sdlc/pi/PI.md` (scope alignment) |
| Feature | PRD + PI + parent epic via `gh issue view #N` (scope consistency) |
| Story | PRD (security/data constraints) + parent feature + parent epic via `gh issue view` |

## Draft Reviewer Agent Design

### Location

`.claude/plugins/sdlc/agents/draft-reviewer/AGENT.md`

### Frontmatter

```yaml
---
name: draft-reviewer
description: Reviews SDLC draft artifacts for completeness, consistency with upstream artifacts, and readiness for sdlc:create execution
tools: Read, Bash, Grep, Glob
---
```

### System Prompt

The agent receives:
- Draft file path
- Artifact level (prd/pi/epic/feature/story)
- Upstream artifact paths/references appropriate to the level

### What It Checks

| Category | What to Look For |
|----------|------------------|
| Completeness | Empty sections, `[TBD]` placeholders, missing required fields per the level's draft template |
| Upstream Consistency | Draft contradicts PRD constraints, PI scope, parent epic/feature body |
| Internal Consistency | Draft sections contradict each other (e.g., success criteria don't match overview) |
| Dependency Validity | Referenced issue numbers exist and are the correct type (via `gh issue view`) |
| Scope | Draft scope matches the level (not a story masquerading as an epic, features match parent epic) |
| YAGNI | Unrequested sections, over-specification for the level |

### What It Does NOT Check

- Stylistic preferences or wording quality
- "Could be more detailed" suggestions
- Creative decisions (that's the user's job in Task 8)
- Anything that's opinion rather than verifiable fact

### Calibration

"Only flag issues that would cause real problems when `sdlc:create` or `sdlc:update` tries to execute this draft. A missing required field, a contradiction with the PRD, or a dependency on a nonexistent issue — those are issues. Minor wording improvements and stylistic preferences are not. Approve unless there are serious gaps."

### Output Format

```
Status: [Approved | Issues Found]

Issues:
1. [Section]: [specific problem and why it matters]
2. [Section]: [specific problem and why it matters]

Recommendations (advisory, not blocking):
- [suggestion]
```

### Loop Behavior

- Dispatched by SKILL.md in Task 7
- If `Issues Found`: SKILL.md fixes the issues in the draft, re-dispatches
- Max 3 iterations
- If still failing after 3: SKILL.md surfaces the remaining issues to the user and lets them decide

## What Changes vs Current State

### SKILL.md
- **Rewritten** from ~390 lines to ~150 lines
- **Added:** TaskCreate/TaskUpdate for 9-task checklist
- **Added:** Agent dispatch for draft reviewer (Task 7)
- **Removed:** Detailed per-phase prose (moved to reference guides as depth-specific branching)
- **Kept:** Hard gate, anti-rationalization, dependency format, new vs reshape detection, integration section

### Reference Guides (all 5)
- **Added:** Pre-flight checklist
- **Added:** Discovery by Depth section (explicit branching per LIGHT/STANDARD/DEEP)
- **Added:** Approaches by Depth section
- **Added:** Review Context section (for draft reviewer agent)
- **Kept:** Scope assessment criteria, question templates, draft body template, level-specific context instructions, all existing content

### New Files
- `agents/draft-reviewer/AGENT.md` — Custom plugin agent for automated draft review

### Plugin Manifest
- `plugin.json` — No changes needed (agents are auto-discovered from the `agents/` directory)

## Out of Scope

- Changing the 5-phase conceptual model (phases map to tasks, not replaced)
- Modifying the depth system (LIGHT/STANDARD/DEEP stays as-is, just better enforced)
- Changing draft format or location (`.claude/sdlc/drafts/` stays)
- Modifying sdlc:create or sdlc:update (they consume drafts the same way)
- Dynamic task generation from guides (deferred — tasks are hardcoded like brainstorming)
- Visual companion (not applicable to SDLC text artifacts)
