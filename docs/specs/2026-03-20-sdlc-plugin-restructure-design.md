# SDLC Plugin Restructure — Design Spec

## Problem

The current SDLC plugin's define skill uses a depth-based system (LIGHT/STANDARD/DEEP) that prescribes exact question counts, question content, and approach counts per artifact level. This makes brainstorming feel rigid and formulaic — like filling out a form, not having a creative conversation. The artifact hierarchy (Epic > Feature > Story) is also inflexible: features always require a stories list, and users must know the artifact level before they start.

The core disconnect: the depth system micromanages the creative process when it should only govern the output format.

## Design Principles

1. **Creative freedom in brainstorming, rigidity in output.** The conversation should feel natural and exploratory. The draft template should be structured and consistent every time.
2. **Each level only plans one layer deep.** PI defines epics as goals. Epics discover features. Features discover stories if needed.
3. **Classification is discovered, not declared.** Users can bring an idea and the skill helps figure out what level it is.
4. **Impact analysis executes decisions, it doesn't make them.** Creative decisions happen during brainstorming. The impact phase inventories those decisions and dispatches agents to carry them out.
5. **One source of truth for operational logic.** Shared reference files, not duplicated logic across skills and agents.
6. **Amended two-phase principle.** The original principle was "define brainstorms, create/update execute — never mix." This redesign amends it: define may dispatch operational agents for **side-effect artifacts** (PI updates, parent issue updates, issue closures) discovered during impact analysis. The **primary artifact** (the one brainstormed) still follows the two-phase flow — define produces the draft, create publishes it. Side-effect execution is scoped to cascading updates that the user confirms one by one; define never silently mutates artifacts.

## Artifact Definitions

### Epic

A business goal or outcome that requires multiple pieces of work to achieve. You know it's an epic because you can't answer "is it done?" without checking multiple things. An epic lives in the PI as a goal with scope seeds (bullet points describing what it should accomplish). Those scope seeds become the starting point for feature discovery later.

*Example: "Plugin manages SDLC lifecycle autonomously"*

### Feature

A concrete chunk of work that moves an epic forward. The primary planning and execution unit. A feature can be:

- **Directly implementable** — small enough to build without further decomposition (labeled `size:small`)
- **Decomposable** — big enough to break into stories during brainstorming (labeled `size:large`)

A feature always belongs to an epic. It has a description of what it does, not how to build it. The "how" is discovered during brainstorming at the feature level.

*Example (directly implementable): "Fix draft-reviewer false positives"*
*Example (decomposable): "Define skill supports creative brainstorming"*

### Story

The smallest implementable unit of work. Always a leaf node, always belongs to a feature. A story exists only when a feature is complex enough to need decomposition. It describes a single task that can be completed in one session with clear acceptance criteria and file scope.

*Example: "Remove depth-based question system from define skill"*

### Key Rule

Each level only plans one layer deep. PI defines epics as goals. Epic brainstorming discovers features. Feature brainstorming discovers stories (if needed). You never plan stories during PI planning.

## Plugin Structure

```
.claude/plugins/sdlc/
  plugin.json
  templates/                        ← shared, used by define + create + update
    epic-template.md
    feature-template.md
    story-template.md
    prd-template.md
    pi-template.md
  skills/
    define/
      SKILL.md                      ← orchestrator
      reference/                    ← brainstorming guides only
        epic-brainstorm.md
        feature-brainstorm.md
        story-brainstorm.md
        prd-brainstorm.md
        pi-brainstorm.md
    create/
      SKILL.md
      reference/                    ← execution references only
        epic-execution.md
        feature-execution.md
        story-execution.md
        prd-execution.md
        pi-execution.md
    update/
      SKILL.md
      reference/                    ← execution references only
        epic-update.md
        feature-update.md
        story-update.md
        prd-update.md
        pi-update.md
    init/
    capture/
    status/
    reconcile/
    retro/
  agents/
    draft-reviewer/                 ← existing
      AGENT.md
    create-agent/                   ← new
      AGENT.md
    update-agent/                   ← new
      AGENT.md
    impact-analysis-agent/          ← new
      AGENT.md
```

### Separation of Concerns

| Location | Contains | Used By |
|----------|----------|---------|
| `templates/` | Rigid output formats (frontmatter, sections, formatting) | define (drafting), create, update, agents |
| `skills/define/reference/` | Brainstorming guides (lightweight checklists) | define only |
| `skills/create/reference/` | Execution references for creating new artifacts (gh commands, validation, procedures) | create skill, create-agent |
| `skills/update/reference/` | Execution references for updating existing artifacts (surgical edits, escalation rules) | update skill, update-agent |

## Define Skill — Orchestrator Design

### Invocation

`/sdlc:define` — level is optional. User can specify (`/sdlc:define epic`) or bring an idea and the skill classifies it.

### Process Flow

1. **Load context** — Read PRD, current PI, and any relevant existing artifacts. Understand the current state of the project.

2. **Creative brainstorming** — Free-flowing conversation guided by an internal checklist from the brainstorming guide for that level. The checklist varies by level but is a reminder of what the skill needs to understand before it can draft — not a script. If the level isn't known yet, the brainstorm discovers it. The skill asks follow-ups based on what's emerging, doesn't march through prescribed questions.

3. **Scope classification** — If the user didn't specify a level, the skill recommends one with reasoning: "Based on what we've discussed, this sounds like a feature under Epic #12." This can happen naturally during the brainstorm, not as a formal gate. If mid-brainstorm the scope grows or shrinks, the skill reclassifies without restarting.

4. **Propose approaches** — 2-3 ways to structure/decompose the work, with trade-offs and a recommendation. Reclassification can also happen here if the approaches reveal the scope is different than expected.

5. **Draft** — Take everything from the brainstorm and pour it into the rigid template for that level. Template comes from `templates/<level>-template.md`. Structured, consistent format every time.

6. **Draft review loop** — Dispatch draft-reviewer agent. Fix issues, re-dispatch (max 3 iterations). If still failing, surface to user.

7. **User reviews draft** — Present full draft. Discuss, revise until explicit approval.

8. **Impact analysis** — The confirm-then-dispatch loop:
   - Dispatch impact-analysis-agent with brainstorm context + current artifact state
   - Agent returns structured list of impacts
   - Present impacts one by one to user
   - Each impact is a mini-conversation — creative, back-and-forth, may involve follow-up questions
   - As each impact is confirmed, dispatch the appropriate operational agent (create-agent or update-agent)
   - Independent updates can run in parallel; dependent updates run sequentially

9. **Announce next step** — "Draft saved. Run `/sdlc:create <level>` when ready, or I can dispatch the create agent now." This refers to the **primary artifact** only — the one brainstormed. Side-effect artifacts (PI updates, parent issue edits, closures) were already dispatched in Step 8 as the user confirmed each one.

### What's Removed from Current Define

- Depth system (LIGHT/STANDARD/DEEP) — all scope assessment matrices, question count prescriptions, approach count rules
- "Discovery by Depth" sections
- "Approaches by Depth" sections
- Fixed question sets per level
- Mandatory level argument

### The Brainstorming Experience

The skill opens by meeting the user where they are. It has an internal checklist of things it needs to understand (from the brainstorming guide) but doesn't march through them. It listens, asks follow-ups based on what's emerging, and checks off items internally as the conversation naturally covers them.

**Internal signals for "I have enough to draft":**
- Can articulate back what the artifact is and why it exists
- Knows where it fits in the hierarchy (which epic, which PI)
- Understands the rough scope (what's in, what's out)
- Has enough to fill the template without inventing details

**Reclassification:** If mid-brainstorm the scope grows or shrinks, the skill adapts naturally: "We started talking about a feature, but this is really its own epic — there are at least three distinct chunks of work here. Want to reframe this as an epic?" The brainstorm continues, it doesn't restart.

### Reshape Flow

Reshaping existing artifacts follows the same process with these adjustments:

**Detection:** Issue number in args (`/sdlc:define epic #12`) = reshape. Keywords like "reshape/rethink/revise/update" in the conversation = reshape. Existing draft in drafts directory = ask user if they're continuing.

**Brainstorming:** The skill loads the existing artifact (from GitHub or git) as context. The conversation explores what needs to change and why. Same creative freedom — no prescribed questions.

**Draft:** Uses the same template as new artifacts, but includes a `## Changes` section:

```markdown
## Changes

| Section | Change | Old Value | New Value |
|---------|--------|-----------|-----------|
| <section> | <what changed> | <old> | <new> |
```

**Impact analysis:** Same confirm-then-dispatch loop, but impacts are likely different (e.g., "Feature #15's parent epic changed scope — does Feature #15 still belong?").

**Announce next step:** "Draft saved with changes documented. Run `/sdlc:update <level> <number>` to apply the changes, or I can dispatch the update agent now."

### Pre-Flight Checks

Before brainstorming can begin, the skill verifies prerequisites exist:

| Level | Prerequisites |
|-------|--------------|
| PRD | Check if `.claude/sdlc/prd/PRD.md` exists (greenfield vs brownfield vs reshape) |
| PI | PRD exists. Check for previous retros. Check for active PI. |
| Epic | PRD exists. PI exists. |
| Feature | PRD exists. PI exists. Parent epic resolvable via `gh issue view`. |
| Story | PRD exists. Parent feature and parent epic resolvable via `gh issue view`. |

If prerequisites are missing, the skill tells the user what needs to exist first and suggests the appropriate `/sdlc:define` invocation.

### Draft File Naming

Drafts are written to `.claude/sdlc/drafts/<level>-<name>.md`. When level is not known at invocation time, the file is created after scope classification (Step 3). For reshapes: `<level>-<issue-number>.md`.

## Brainstorming Guides

Lightweight internal checklists, one per level. They tell the skill "before you can draft this artifact, make sure you understand these things." The skill weaves them into natural conversation — they're a pilot's checklist, not a script.

### PRD Brainstorming Guide

```
Before drafting a PRD, make sure you understand:
- What is this project and why does it exist?
- What's the tech stack?
- What's the high-level architecture?
- What are the key data models?
- What are the API contracts or interfaces?
- What are the security constraints?
- What's the roadmap (high-level phases)?
- What's in scope and out of scope?
- What decisions have already been made and why?
For brownfield projects: dispatch a research agent to analyze the existing codebase first.
```

### PI Brainstorming Guide

```
Before drafting a PI, make sure you understand:
- What's the theme for this increment? (one-line summary of focus)
- What epics should be tackled? (goals, not implementation details)
- What does success look like for each epic?
- What are the rough scope seeds for each epic? (bullet points, not features)
- Are there cross-epic dependencies?
- Which epics can be parallelized (worktree strategy)?
- What's the timeline?
- Any carry-over from previous PI?
```

### Epic Brainstorming Guide

```
Before drafting an epic, make sure you understand:
- What's the goal/outcome?
- How do we know it's done? (success criteria)
- What are the rough chunks of work? (feature seeds)
- What's the priority relative to other epics?
- Any dependencies on other epics?
- What areas of the codebase does this touch?
```

### Feature Brainstorming Guide

```
Before drafting a feature, make sure you understand:
- What does this feature do? (description, not implementation)
- What's the acceptance criteria?
- Is this directly implementable (size:small) or does it need decomposition (size:large)?
- If large: what are the stories?
- What's NOT in scope for this feature?
- Any dependencies on other features or stories?
- Which parent epic does this belong to?
```

### Story Brainstorming Guide

```
Before drafting a story, make sure you understand:
- What's the single task to accomplish?
- What are the acceptance criteria? (specific, testable)
- What files will be created or modified?
- Are there existing patterns to follow?
- Any technical considerations or edge cases?
- What blocks this or what does this block?
- Which parent feature and epic does this belong to?
For stories with unclear file scope: dispatch a research agent to find relevant files and patterns.
```

The skill doesn't ask these as a list. It checks them off internally as the conversation covers each point. Some might get answered without being asked directly.

## Draft Templates

Rigid output formats. Exact frontmatter fields, exact section headings, exact formatting. Shared by define (during drafting), create, and update.

### PI Template

```yaml
---
name: PI-<N>
theme: <one-line theme>
started: YYYY-MM-DD
target: YYYY-MM-DD
status: active
---

## Goals
<What this PI aims to achieve overall>

## Epics

### Epic: <name>
**Goal:** <what success looks like — one or two sentences>
**Priority:** critical | high | medium | low
**Scope seeds:**
- <bullet point — rough chunk of work>
- <bullet point — these become feature candidates later>
- <bullet point — not commitments, just direction>

## Dependencies
<Cross-epic dependencies only>

## Worktree Strategy
<Which epics can be parallelized>
```

### Epic Template

```yaml
---
type: epic
name: <epic name>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
---

## Overview
## Success Criteria
## Features
## Non-goals
## Dependencies
```

### Feature Template

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

## Description
## Acceptance Criteria
## Stories (only if size: large — omit entirely if size: small)
## Non-goals
## Dependencies
## Parent
```

### Story Template

```yaml
---
type: story
name: <story name>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
parent-epic: <issue number>
parent-feature: <issue number>
---

## Description
## Acceptance Criteria
## File Scope
## Technical Notes
## Dependencies
## Parent
```

### PRD Template

```yaml
---
name: <project name>
version: 1.0
created: YYYY-MM-DD
status: draft
---

## Overview
## Tech Stack
## Architecture
## Data Models
## API Contracts
## Security Constraints
## Roadmap
## Acceptance Criteria
## Out of Scope
## Label Taxonomy
## Decision Log
```

## Agent Architecture

### impact-analysis-agent

Dispatched by define during the impact phase.

**Input (passed via agent prompt):**
- Summary of brainstorm decisions: what artifact was defined, at what level, key scope decisions
- Any reclassifications that happened during brainstorming
- The draft file path
- Current PI path, relevant epic/feature issue numbers

**Output (returned to define):**
```
Impacts found: N

1. [category] [target]: [description]
   - Category: pi-update | epic-update | feature-update | story-update | prd-update | issue-closure | new-artifact
   - Target: file path or issue number
   - Description: what needs to change and why

2. [category] [target]: [description]
   ...
```

**Impact categories the agent scans for:**
- PI updates (new epic added, epic scope changed, epic removed/deferred)
- Parent issue updates (new child added, child moved, scope description changed)
- Sibling issue updates (dependency changes affecting siblings)
- PRD updates (roadmap changes, acceptance criteria changes, decision log entries)
- Issue closures (existing issues superseded or addressed by new artifact)
- New artifacts needed (brainstorm revealed work that needs its own define cycle)

### create-agent

Dispatched by define or the impact analysis loop to create new artifacts.

**Input:** Draft file path, artifact level.

**Behavior:** Loads the template from `${CLAUDE_PLUGIN_ROOT}/templates/<level>-template.md` and the execution reference from `${CLAUDE_PLUGIN_ROOT}/skills/create/reference/<level>-execution.md`. Follows the execution reference exactly. Does NOT ask for user confirmation (the user already confirmed during impact analysis). Does NOT offer to delete the draft (define manages draft lifecycle). Does NOT run cascade logic (that's the impact analysis loop's job).

**Output:** Created artifact identifier (issue number + URL for GitHub issues, file path for git artifacts).

**Difference from `/sdlc:create` skill:** The skill has user-facing gates (confirmation, draft cleanup offer, cascade reporting). The agent skips these because define orchestrates those interactions.

### update-agent

Dispatched by define or the impact analysis loop to update existing artifacts.

**Input:** Target (issue number or file path), specific change to make (structured: which section, old value, new value).

**Behavior:** Loads the execution reference from `${CLAUDE_PLUGIN_ROOT}/skills/update/reference/<level>-execution.md`. Makes the surgical edit. Does NOT ask for user confirmation (already confirmed). Does NOT show side-by-side comparison (define handled that in the impact discussion). Does NOT escalate to define (it's already in define).

**Output:** Confirmation of change made.

**Difference from `/sdlc:update` skill:** The skill has escalation logic (3+ changes → redirect to define) and user confirmation. The agent receives pre-approved, scoped changes from define.

### draft-reviewer (existing, updated path)

Validates drafts before user review. Checks completeness, upstream consistency, internal consistency, dependency validity, scope, and YAGNI. Max 3 iterations before escalating to human.

**Updated:** Must read draft templates from `${CLAUDE_PLUGIN_ROOT}/templates/<level>-template.md` (previously read from define's reference guides). The required fields for validation come from the template, not the brainstorming guide.

### update skill reference directory

The update skill keeps its own `reference/` directory with its execution references (currently `epic-update.md`, `feature-update.md`, etc.). These are distinct from create's execution references — create handles new artifact creation, update handles surgical edits to existing artifacts. The update-agent loads from `skills/update/reference/`, not from create's references.

### Dispatch Strategy

Sequential-with-parallelism (like superpowers subagent-driven-development):
- Analysis agent runs first (needs full context)
- For each confirmed impact, dispatch the appropriate agent
- Independent updates can run in parallel using Claude Code's Agent tool (multiple agent dispatches in a single message)
- Dependent updates run sequentially (create epic first, then update PI with its number)

**Parallelism note:** Claude Code's Agent tool supports dispatching multiple agents in a single message for parallel execution. The define skill should use this when two confirmed impacts are independent (e.g., updating two unrelated epics). When impacts have dependencies (e.g., creating a new issue and then referencing its number in a PI update), they must be sequential.

## Label Taxonomy Updates

**Kept as-is:**
- `type:epic`, `type:feature`, `type:story`, `type:spike`, `type:bug`, `type:chore`
- `status:todo`, `status:in-progress`, `status:done`, `status:blocked`
- `priority:critical`, `priority:high`, `priority:medium`, `priority:low`
- `area:*` labels
- `triage` label

**Added:**
- `size:small` — feature is directly implementable, no child stories
- `size:large` — feature is decomposed into stories

**Validation rules:**
- `size:*` labels apply ONLY to `type:feature` issues. The reconcile skill should strip `size:*` labels from non-feature issues.
- Create and update should validate that size labels are only set on features.
- A `size:large` feature must have at least one child story. A `size:small` feature must have zero child stories.

## What Gets Removed

- Depth system (LIGHT/STANDARD/DEEP) — all scope assessment matrices, question count prescriptions, approach count rules
- "Discovery by Depth" sections in all reference guides
- "Approaches by Depth" sections in all reference guides
- Fixed question sets per level
- Mandatory level argument on `/sdlc:define`
- Current monolithic reference files (replaced by brainstorming guides + shared templates)
- Feature template requiring stories list (stories section becomes optional)
- Heavy PI template with full feature lists (replaced by lighter scope-seeds format)
- Flat epics (stories directly under an epic, no features) — the new model requires Epic > Feature > optional Story. What was previously a "flat epic with stories" becomes an epic with `size:small` features (each directly implementable). This simplifies the hierarchy to one consistent pattern.

## What Gets Kept

- Two-phase principle (creative → execution) — amended per Design Principle #6: primary artifact follows two-phase, side-effects dispatch during impact analysis
- Draft-reviewer agent (updated to read templates from new location)
- Execution references under create/update (updated: create references must handle `size:small` features without `## Stories` section; update references unchanged)
- All operational skills as user-invocable slash commands
- Drafts directory workflow (`.claude/sdlc/drafts/`)
- Conventional commit format
- Bidirectional dependency format (`- Blocked by: #N` / `- Blocks: #N`)
- New vs reshape detection
- Canonical dependency format

## What's New

- Three agents: impact-analysis-agent, create-agent, update-agent
- Shared `templates/` directory at plugin level
- Brainstorming guides (lightweight checklists) replacing depth-driven question scripts
- Level-optional invocation of define
- Impact analysis confirm-then-dispatch loop
- `size:small` / `size:large` labels for features
- Lighter PI format with scope seeds instead of feature lists

## End-to-End Example

User invokes `/sdlc:define` with: "I want to rethink how the define skill works — it's too rigid"

**Brainstorming:** Skill loads context (PRD, PI, existing epics). Opens naturally: "Tell me more about what feels rigid." Conversation follows — user discusses creative freedom, depth system problems, artifact definitions. Skill tracks internally: goal is clear, scope is growing, touches multiple areas. Eventually: "This is shaping up to be an epic — you're talking about rewriting define, restructuring reference files, creating new agents, and updating the PI format. Agree?" User agrees. Brainstorm discovers feature candidates.

**Propose approaches:** 2-3 ways to sequence the work. User picks one.

**Draft:** Skill pulls up `epic-template.md`, fills it with everything from the brainstorm. Structured, consistent.

**Draft review:** Draft-reviewer agent validates. User reviews, tweaks, approves.

**Impact analysis:**
- Analysis agent dispatched: reads brainstorm context + current PI + existing artifacts. Returns: "3 impacts found"
- Impact 1: "PI needs this new epic added." → User: "Yes" → Dispatch update-agent → updates PI.md
- Impact 2: "Issue #7 is addressed by this epic. Close it?" → User: "Yes" → Dispatch update-agent → closes #7
- Impact 3: "PRD acceptance criteria may need updating. Flag for future reshape?" → User: "Not now" → No dispatch
- "All impacts resolved. Draft saved. Run `/sdlc:create epic` when ready."

## End-to-End Example: Feature Under Existing Epic

User invokes `/sdlc:define` with: "I need to add error handling to the create skill"

**Brainstorming:** Skill loads context — sees Epic #12 (SDLC Plugin Restructure) exists with relevant features. Opens naturally: "What kind of errors are you running into?" User explains: validation failures when drafts are malformed, gh CLI errors, missing prerequisites. Skill tracks internally: this is scoped work under an existing epic, probably a feature. "This sounds like a feature under Epic #12 — concrete work with clear boundaries. Does that feel right?" User agrees. Skill explores: is this directly implementable or does it break down? User describes three distinct areas (validation errors, CLI errors, prerequisite checks). Skill: "That's three distinct areas — I'd call this `size:large` with three stories. Agree?"

**Draft:** Feature template with `size:large`, three stories listed. User reviews and approves.

**Impact analysis:**
- Analysis agent: "1 impact found"
- Impact 1: "Epic #12 needs this feature added to its Features checklist." → User: "Yes" → Dispatch update-agent → updates Epic #12

## End-to-End Example: Reclassification During Brainstorm

User invokes `/sdlc:define` with: "I want to add a size label to features"

**Brainstorming:** Skill loads context. Opens: "Tell me more about what the size label would do." User explains small vs large features. Skill initially thinks this is a story — a single label addition. But as the conversation goes deeper, user describes: label taxonomy changes, reconcile skill updates, create/update validation changes, template changes. Skill: "We started talking about a small change, but this touches labels, reconcile, create, update, and templates — that's multiple features across different areas. This is really an epic. Want to reframe?" User agrees. Brainstorm continues at epic level, discovering features.

## Migration Steps

When this spec is implemented, the following existing artifacts need updating:

1. **PRD Decision Log** — The entry for "Depth-based discovery (LIGHT/STANDARD/DEEP)" should be superseded with a new entry: "Creative brainstorming with lightweight checklists — replaced depth system to enable creative freedom while maintaining structured output."
2. **PRD Acceptance Criteria** — Update to reflect new artifact definitions (flexible hierarchy, optional stories).
3. **PRD Label Taxonomy** — Add `size:small` and `size:large` with validation rules.
4. **CLAUDE.md** — Update the "Two-phase workflow" convention to reflect the amended principle (Design Principle #6). Update artifact hierarchy description.
5. **Existing PI** (if one exists) — Migrate to the lighter scope-seeds format.

## Relationship to Previous Spec

This spec supersedes `2026-03-20-define-skill-overhaul-design.md`, which reinforced the depth-based system. That spec restructured how depth was enforced; this spec removes depth entirely and replaces it with creative brainstorming guided by lightweight checklists.
