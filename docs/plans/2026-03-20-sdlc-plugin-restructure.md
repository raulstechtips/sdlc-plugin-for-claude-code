# SDLC Plugin Restructure Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure the SDLC plugin to replace the rigid depth-based system with creative brainstorming, flexible artifact hierarchy (optional stories), shared templates, and operational dispatch agents.

**Architecture:** Define becomes the orchestrator with creative brainstorming guided by lightweight checklists. Templates are extracted to a shared directory. Three new agents (impact-analysis, create-agent, update-agent) handle operational work dispatched by define. The artifact model is simplified: Epic > Feature > optional Story.

**Tech Stack:** Claude Code plugin system (markdown skills/agents with YAML frontmatter), `gh` CLI, `git`, `bash`

**Spec:** `docs/specs/2026-03-20-sdlc-plugin-restructure-design.md`

---

## File Structure

### New Files to Create
- `templates/prd-template.md` — PRD draft template (rigid output format)
- `templates/pi-template.md` — PI draft template
- `templates/epic-template.md` — Epic draft template
- `templates/feature-template.md` — Feature draft template (with conditional Stories section)
- `templates/story-template.md` — Story draft template
- `skills/define/reference/prd-brainstorm.md` — PRD brainstorming guide (lightweight checklist)
- `skills/define/reference/pi-brainstorm.md` — PI brainstorming guide
- `skills/define/reference/epic-brainstorm.md` — Epic brainstorming guide
- `skills/define/reference/feature-brainstorm.md` — Feature brainstorming guide
- `skills/define/reference/story-brainstorm.md` — Story brainstorming guide
- `agents/impact-analysis-agent/AGENT.md` — Impact analysis agent
- `agents/create-agent/AGENT.md` — Operational create agent
- `agents/update-agent/AGENT.md` — Operational update agent

### Files to Rewrite
- `skills/define/SKILL.md` — Complete rewrite as orchestrator

### Files to Modify
- `skills/create/reference/feature-execution.md` — Add `size` field support, make Stories optional
- `skills/create/reference/epic-execution.md` — Remove flat epic (story stubs) path, epics always have features
- `skills/create/reference/story-execution.md` — Remove flat epic `parent-feature: none` path
- `skills/update/reference/feature-update.md` — Add size label change examples
- `skills/update/reference/story-update.md` — Remove flat epic references
- `skills/reconcile/SKILL.md` — Remove flat epic exception, add size label validation check
- `agents/draft-reviewer/AGENT.md` — Update template lookup path, add size validation
- `skills/init/SKILL.md` — Add `size:small` and `size:large` label creation
- `plugin.json` — Bump version

### Files to Modify (Project Artifacts)
- `.claude/sdlc/prd/PRD.md` — Update artifact definitions, label taxonomy, decision log
- `CLAUDE.md` — Update conventions, project structure, workflow description

### Files to Delete
- `skills/define/reference/prd-guide.md` — Replaced by prd-brainstorm.md + templates/prd-template.md
- `skills/define/reference/pi-guide.md` — Replaced by pi-brainstorm.md + templates/pi-template.md
- `skills/define/reference/epic-guide.md` — Replaced by epic-brainstorm.md + templates/epic-template.md
- `skills/define/reference/feature-guide.md` — Replaced by feature-brainstorm.md + templates/feature-template.md
- `skills/define/reference/story-guide.md` — Replaced by story-brainstorm.md + templates/story-template.md

All paths below are relative to `.claude/plugins/sdlc/` unless they start with `.claude/sdlc/` or `CLAUDE.md`.

---

### Task 1: Create Shared Templates Directory

Create the 5 template files that define rigid output formats for each artifact level. These are the single source of truth for draft structure — used by define (drafting), create (validation/execution), update, and agents.

**Files:**
- Create: `templates/prd-template.md`
- Create: `templates/pi-template.md`
- Create: `templates/epic-template.md`
- Create: `templates/feature-template.md`
- Create: `templates/story-template.md`

- [ ] **Step 1: Create templates directory**

```bash
mkdir -p .claude/plugins/sdlc/templates
```

- [ ] **Step 2: Write prd-template.md**

```markdown
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
```

- [ ] **Step 3: Write pi-template.md**

```markdown
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
```

- [ ] **Step 4: Write epic-template.md**

```markdown
# Epic Template

The canonical output format for Epic drafts. Epics are GitHub Issues with `type:epic` label.

## Frontmatter

```yaml
---
type: epic
name: <epic name>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
---
```

## Required Sections

- `## Overview` — Goal/outcome, value proposition
- `## Success Criteria` — Checklist of measurable conditions for "done"
- `## Features` — Checklist of features with `(#TBD)` placeholders for issue numbers
- `## Non-goals` — Explicit exclusions
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)
```

- [ ] **Step 5: Write feature-template.md**

```markdown
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
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)
- `## Parent` — `Epic: #<parent-epic>`

## Conditional Sections

- `## Stories` — **Only if `size: large`**. Checklist of stories with `(#TBD)` placeholders. **Omit entirely if `size: small`.**

## Validation Rules

- `size:small` features MUST NOT have a `## Stories` section
- `size:large` features MUST have a `## Stories` section with at least one item
```

- [ ] **Step 6: Write story-template.md**

```markdown
# Story Template

The canonical output format for Story drafts. Stories are GitHub Issues with `type:story` label. Stories are always leaf nodes — they belong to a feature and have no children.

## Frontmatter

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
```

## Required Sections

- `## Description` — What single task to accomplish
- `## Acceptance Criteria` — Checklist of specific, testable conditions
- `## File Scope` — Lists of files to create and modify
- `## Technical Notes` — Implementation considerations, patterns to follow
- `## Dependencies` — Bidirectional format: `- Blocked by: #N` / `- Blocks: #N` (or `none`)
- `## Parent` — `Epic: #<parent-epic>, Feature: #<parent-feature>`
```

- [ ] **Step 7: Commit**

```bash
git add .claude/plugins/sdlc/templates/
git commit -m "feat(sdlc): create shared templates directory with 5 artifact templates"
```

---

### Task 2: Create Brainstorming Guides

Replace the monolithic depth-based reference guides with lightweight brainstorming checklists. These guide the creative conversation without prescribing questions.

**Files:**
- Create: `skills/define/reference/prd-brainstorm.md`
- Create: `skills/define/reference/pi-brainstorm.md`
- Create: `skills/define/reference/epic-brainstorm.md`
- Create: `skills/define/reference/feature-brainstorm.md`
- Create: `skills/define/reference/story-brainstorm.md`
- Delete: `skills/define/reference/prd-guide.md`
- Delete: `skills/define/reference/pi-guide.md`
- Delete: `skills/define/reference/epic-guide.md`
- Delete: `skills/define/reference/feature-guide.md`
- Delete: `skills/define/reference/story-guide.md`

- [ ] **Step 1: Write prd-brainstorm.md**

```markdown
# PRD Brainstorming Guide

Internal checklist for the define skill. Weave these into natural conversation — do NOT ask them as a list.

## Before you can draft, make sure you understand:

- What is this project and why does it exist?
- What's the tech stack?
- What's the high-level architecture?
- What are the key data models?
- What are the API contracts or interfaces?
- What are the security constraints?
- What's the roadmap (high-level phases)?
- What's in scope and out of scope?
- What decisions have already been made and why?

## Special handling

**Brownfield projects:** Before asking questions, dispatch a research agent to analyze the existing codebase:
- Tech stack (languages, frameworks, package managers)
- Directory structure (top-level layout, key directories)
- Architecture patterns
- Data models
- API endpoints
- Security patterns

Use the research findings to pre-fill what you can and focus questions on what the codebase can't tell you (goals, roadmap, constraints, decisions).

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/prd-template.md`
```

- [ ] **Step 2: Write pi-brainstorm.md**

```markdown
# PI Brainstorming Guide

Internal checklist for the define skill. Weave these into natural conversation — do NOT ask them as a list.

## Before you can draft, make sure you understand:

- What's the theme for this increment? (one-line summary of focus)
- What epics should be tackled? (goals, not implementation details)
- What does success look like for each epic?
- What are the rough scope seeds for each epic? (bullet points, not features)
- Are there cross-epic dependencies?
- Which epics can be parallelized (worktree strategy)?
- What's the timeline?
- Any carry-over from previous PI?

## Context to load

- Read `.claude/sdlc/prd/PRD.md` — especially Roadmap and Acceptance Criteria sections
- Check `.claude/sdlc/retros/` for previous retrospectives
- Check `.claude/sdlc/pi/completed/` for previous PI plans
- Check if an active PI exists at `.claude/sdlc/pi/PI.md`

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/pi-template.md`
```

- [ ] **Step 3: Write epic-brainstorm.md**

```markdown
# Epic Brainstorming Guide

Internal checklist for the define skill. Weave these into natural conversation — do NOT ask them as a list.

## Before you can draft, make sure you understand:

- What's the goal/outcome?
- How do we know it's done? (success criteria)
- What are the rough chunks of work? (feature seeds)
- What's the priority relative to other epics?
- Any dependencies on other epics?
- What areas of the codebase does this touch?

## Context to load

- Read `.claude/sdlc/pi/PI.md` — find the epic's scope seeds if it was planned in the PI
- Read `.claude/sdlc/prd/PRD.md` — check Roadmap and Architecture sections
- Check existing epics: `gh issue list --label "type:epic" --state open --json number,title`

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/epic-template.md`
```

- [ ] **Step 4: Write feature-brainstorm.md**

```markdown
# Feature Brainstorming Guide

Internal checklist for the define skill. Weave these into natural conversation — do NOT ask them as a list.

## Before you can draft, make sure you understand:

- What does this feature do? (description, not implementation)
- What's the acceptance criteria?
- Is this directly implementable (size:small) or does it need decomposition (size:large)?
- If large: what are the stories?
- What's NOT in scope for this feature?
- Any dependencies on other features or stories?
- Which parent epic does this belong to?

## Context to load

- Read the parent epic: `gh issue view <parent-epic> --json title,body,labels`
- Read `.claude/sdlc/pi/PI.md` — check if this feature was listed as a scope seed
- Read `.claude/sdlc/prd/PRD.md` — check relevant Architecture and Data Models sections

## Size classification

- **size:small** — Directly implementable. No stories needed. One person, one or two sessions. Clear file scope.
- **size:large** — Needs decomposition into stories. Multiple sessions, multiple areas, or multiple people.

The brainstorm should naturally discover which size fits. Don't force the classification — let it emerge from the conversation.

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/feature-template.md`
```

- [ ] **Step 5: Write story-brainstorm.md**

```markdown
# Story Brainstorming Guide

Internal checklist for the define skill. Weave these into natural conversation — do NOT ask them as a list.

## Before you can draft, make sure you understand:

- What's the single task to accomplish?
- What are the acceptance criteria? (specific, testable)
- What files will be created or modified?
- Are there existing patterns to follow?
- Any technical considerations or edge cases?
- What blocks this or what does this block?
- Which parent feature and epic does this belong to?

## Context to load

- Read the parent feature: `gh issue view <parent-feature> --json title,body,labels`
- Read the parent epic: `gh issue view <parent-epic> --json title,body,labels`
- Read `.claude/sdlc/prd/PRD.md` — check Security Constraints and Architecture sections

## Research agent

For stories with unclear file scope, dispatch a research agent:
- Find files that will be modified (exact paths)
- Identify existing patterns to follow
- Check for related tests
- Note utilities/helpers to reuse

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/story-template.md`
```

- [ ] **Step 6: Delete old reference guides**

```bash
rm .claude/plugins/sdlc/skills/define/reference/prd-guide.md
rm .claude/plugins/sdlc/skills/define/reference/pi-guide.md
rm .claude/plugins/sdlc/skills/define/reference/epic-guide.md
rm .claude/plugins/sdlc/skills/define/reference/feature-guide.md
rm .claude/plugins/sdlc/skills/define/reference/story-guide.md
```

- [ ] **Step 7: Commit**

```bash
git add .claude/plugins/sdlc/skills/define/reference/
git commit -m "feat(sdlc): replace depth-based guides with brainstorming checklists

Remove 5 monolithic reference guides (prd/pi/epic/feature/story-guide.md)
containing depth matrices, prescribed questions, and approach counts.
Replace with lightweight brainstorming guides that list what the skill
needs to understand without prescribing how to ask."
```

---

### Task 3: Rewrite Define Skill as Orchestrator

Complete rewrite of `skills/define/SKILL.md`. Removes depth system, adds level-optional invocation, creative brainstorming, scope classification, and impact analysis phases.

**Files:**
- Rewrite: `skills/define/SKILL.md`

- [ ] **Step 1: Read the current define skill for reference**

```
Read .claude/plugins/sdlc/skills/define/SKILL.md
```

- [ ] **Step 2: Write the new define SKILL.md**

Write the following content to `skills/define/SKILL.md`:

```markdown
---
name: define
description: Use when defining new SDLC artifacts (PRD, PI, epic, feature, story) or reshaping existing ones through collaborative brainstorming that produces a reviewable local draft.
allowed-tools: Read, Edit, Write, Bash, Grep, Glob, Agent
argument-hint: "[level] [identifier]"
---

I'm using the sdlc:define skill to define/reshape an SDLC artifact.

<HARD-GATE>
Do NOT produce a draft without completing all phases in order. Do NOT skip a phase because the artifact "seems simple." Do NOT present multiple phases at once — complete each one before moving to the next.
</HARD-GATE>

## Red Flags

| Thought | Reality |
|---------|---------|
| "This is simple, skip to the draft" | All phases are mandatory. Simple artifacts have shorter brainstorms, not skipped phases. |
| "I already know what to build" | Discovery catches assumptions. Even if you're right, the user confirms. |
| "Let me skip the approaches phase" | Even simple work benefits from a quick "here's how I'd approach this — agree?" |
| "The draft is obvious, no review needed" | The draft reviewer catches issues you miss after a long conversation. Always run it. |
| "I'll skip impact analysis, nothing changed" | New artifacts always affect something. Even a small story might update a parent feature's checklist. |
| "The user didn't specify a level, I'll guess" | Ask. Propose with reasoning. Don't assume. |
| "I'll combine brainstorming and drafting" | Creative exploration and structured output are separate phases. Finish brainstorming before drafting. |
| "This impact is obvious, no need to discuss" | Every impact is presented to the user. Even obvious ones get confirmed. |

## Process Flow

You MUST create a task for each of these phases and complete them in order:

### Phase 1: Load Context

Parse `$ARGUMENTS` for an optional level and optional identifier.

- If level provided: load brainstorming guide from `${CLAUDE_PLUGIN_ROOT}/skills/define/reference/<level>-brainstorm.md`
- If no level: proceed without a guide — the brainstorm will discover the level
- Read `.claude/sdlc/prd/PRD.md` for project context
- Read `.claude/sdlc/pi/PI.md` if it exists
- Check pre-flight requirements (see Pre-Flight Checks below)
- Detect new vs reshape: issue number in args = reshape; "reshape/rethink/revise/update" keywords = reshape; existing draft in drafts dir = ask user

### Phase 2: Creative Brainstorming

Open-ended conversation guided by the brainstorming guide's internal checklist. The guide lists what you need to understand before drafting — NOT a script of questions to ask.

**How to brainstorm:**
- Meet the user where they are. Start with what they told you.
- Ask follow-up questions based on what's emerging from the conversation
- One question at a time. Wait for the answer before asking the next.
- Check off checklist items internally as the conversation covers them
- Some items may get answered without being asked directly
- If a research agent would help (brownfield PRD, unclear story scope), dispatch one per the brainstorming guide's instructions

**Internal signals for "I have enough to draft":**
- Can articulate back what the artifact is and why it exists
- Knows where it fits in the hierarchy (which epic, which PI)
- Understands the rough scope (what's in, what's out)
- Has enough to fill the template without inventing details

### Phase 3: Scope Classification

If the user didn't specify a level, recommend one with reasoning:

> "Based on what we've discussed, this sounds like a feature under Epic #12 — it's a concrete chunk of work, not a whole new goal. Does that feel right?"

This can happen naturally during brainstorming — it doesn't have to be a formal gate. If mid-brainstorm the scope grows or shrinks, reclassify without restarting:

> "We started talking about a feature, but this is really its own epic — there are at least three distinct chunks of work here. Want to reframe this as an epic?"

Once the level is known, load the brainstorming guide if not already loaded.

### Phase 4: Propose Approaches

Present 2-3 ways to structure or decompose the work. Include trade-offs and your recommendation. Reclassification can also happen here if the approaches reveal the scope is different than expected.

Wait for the user's choice before proceeding.

### Phase 5: Draft

Load the template from `${CLAUDE_PLUGIN_ROOT}/templates/<level>-template.md`. Pour everything from the brainstorm into the rigid format. Structured, consistent output every time.

Write to `.claude/sdlc/drafts/<level>-<name>.md` (new) or `.claude/sdlc/drafts/<level>-<issue-number>.md` (reshape). Ensure the drafts directory exists.

For reshapes, include a `## Changes` section at the end:

| Section | Change | Old Value | New Value |
|---------|--------|-----------|-----------|

### Phase 6: Draft Review Loop

Dispatch the `draft-reviewer` agent with:
- Draft file path
- Artifact level
- Upstream artifact paths (PRD path, PI path, parent issue numbers as applicable)

Fix any issues the agent finds. Re-dispatch (max 3 iterations). If still failing after 3, surface remaining issues to the user.

### Phase 7: User Reviews Draft

Present the full draft. Do not summarize — show everything. Ask: "Want to change anything?" Loop until explicit approval. Do NOT interpret silence or ambiguity as approval.

### Phase 8: Impact Analysis

The confirm-then-dispatch loop:

1. Dispatch `impact-analysis-agent` with: summary of brainstorm decisions, reclassifications, draft file path, current PI path, relevant issue numbers
2. Agent returns structured list of impacts
3. Present impacts to the user ONE AT A TIME
4. Each impact is a mini-conversation — creative, back-and-forth, may involve follow-up questions
5. As each impact is confirmed, dispatch the appropriate operational agent (`create-agent` or `update-agent`)
6. Independent updates can run in parallel (multiple Agent dispatches in one message); dependent updates run sequentially (create issue first, then reference its number)

### Phase 9: Announce Next Step

For new artifacts:
> "Draft saved to `<path>`. Run `/sdlc:create <level>` when ready to push it live, or I can dispatch the create agent now."

For reshapes:
> "Draft saved with changes documented. Run `/sdlc:update <level> <number>` to apply the changes, or I can dispatch the update agent now."

This refers to the **primary artifact** only. Side-effect artifacts were already dispatched in Phase 8.

## Pre-Flight Checks

| Level | Prerequisites |
|-------|--------------|
| PRD | Check if `.claude/sdlc/prd/PRD.md` exists (greenfield vs brownfield vs reshape) |
| PI | PRD exists. Check for previous retros. Check for active PI. |
| Epic | PRD exists. PI exists. |
| Feature | PRD exists. PI exists. Parent epic resolvable via `gh issue view`. |
| Story | PRD exists. Parent feature and parent epic resolvable via `gh issue view`. |

If prerequisites are missing, tell the user what needs to exist first and suggest the appropriate `/sdlc:define` invocation.

## Dependency Format

All dependency references in drafts use this exact format:

- Blocked by: #N, #M
- Blocks: #N, #M

Rules: dash-prefixed, `#` prefix on issue numbers, comma-space separated, `none` when empty. Bidirectional — reconcile enforces consistency.

## Integration

| Scenario | Flow |
|----------|------|
| New artifact | define produces draft → `/sdlc:create` or create-agent pushes to GitHub/git |
| Reshape existing | define produces draft with Changes section → `/sdlc:update` or update-agent applies edits |
| Side-effect updates | Impact analysis dispatches create-agent/update-agent for confirmed cascading changes |
```

- [ ] **Step 3: Verify the new file loads correctly**

```bash
cat .claude/plugins/sdlc/skills/define/SKILL.md | head -10
```

Expected: YAML frontmatter with `name: define`

- [ ] **Step 4: Commit**

```bash
git add .claude/plugins/sdlc/skills/define/SKILL.md
git commit -m "feat(sdlc): rewrite define skill as creative orchestrator

Remove depth system (LIGHT/STANDARD/DEEP). Replace with free-form
brainstorming guided by internal checklists. Add level-optional
invocation, scope classification, impact analysis with agent dispatch.
Define now orchestrates the full lifecycle from idea to artifact."
```

---

### Task 4: Create Three New Agents

Create impact-analysis-agent, create-agent, and update-agent with fully specified contracts.

**Files:**
- Create: `agents/impact-analysis-agent/AGENT.md`
- Create: `agents/create-agent/AGENT.md`
- Create: `agents/update-agent/AGENT.md`

- [ ] **Step 1: Create agent directories**

```bash
mkdir -p .claude/plugins/sdlc/agents/impact-analysis-agent
mkdir -p .claude/plugins/sdlc/agents/create-agent
mkdir -p .claude/plugins/sdlc/agents/update-agent
```

- [ ] **Step 2: Write impact-analysis-agent AGENT.md**

Must include:
- Frontmatter: `name: impact-analysis-agent`, `description: Analyzes brainstorm context to identify cascading impacts on existing SDLC artifacts`, `tools: Read, Bash, Grep, Glob`
- Input contract: brainstorm summary, reclassifications, draft file path, PI path, epic/feature issue numbers
- Process: read draft, read current PI, read relevant issues, scan for each impact category
- Impact categories: pi-update, epic-update, feature-update, story-update, prd-update, issue-closure, new-artifact
- Output format: numbered impacts with category, target, description
- Calibration: only flag impacts that require actual changes, not speculative ones

- [ ] **Step 3: Write create-agent AGENT.md**

Must include:
- Frontmatter: `name: create-agent`, `description: Creates SDLC artifacts from drafts — dispatched by define or impact analysis, not user-invoked`, `tools: Read, Edit, Write, Bash, Grep, Glob`
- Input contract: draft file path, artifact level
- Behavior: load template from `${CLAUDE_PLUGIN_ROOT}/templates/<level>-template.md`, load execution reference from `${CLAUDE_PLUGIN_ROOT}/skills/create/reference/<level>-execution.md`, follow exactly
- What it skips (vs the skill): no user confirmation, no draft cleanup offer, no cascade logic
- Output: created artifact identifier (issue number + URL, or file path)

- [ ] **Step 4: Write update-agent AGENT.md**

Must include:
- Frontmatter: `name: update-agent`, `description: Updates existing SDLC artifacts — dispatched by define or impact analysis, not user-invoked`, `tools: Read, Edit, Write, Bash, Grep, Glob`
- Input contract: target (issue number or file path), specific change (section, old value, new value)
- Behavior: load execution reference from `${CLAUDE_PLUGIN_ROOT}/skills/update/reference/<level>-execution.md`, make surgical edit
- What it skips (vs the skill): no user confirmation, no side-by-side comparison, no escalation to define
- Output: confirmation of change

- [ ] **Step 5: Commit**

```bash
git add .claude/plugins/sdlc/agents/
git commit -m "feat(sdlc): add impact-analysis, create, and update agents

Three operational agents dispatched by define during impact analysis.
Each has a specified input/output contract and references the same
execution files as their skill counterparts."
```

---

### Task 5: Update Draft-Reviewer Agent

Update the template lookup path and add size field validation for features.

**Files:**
- Modify: `agents/draft-reviewer/AGENT.md`

- [ ] **Step 1: Read the current draft-reviewer**

```
Read .claude/plugins/sdlc/agents/draft-reviewer/AGENT.md
```

- [ ] **Step 2: Update Process step 2**

Change line 42 from:
```
2. Read the reference guide for the level at `.claude/plugins/sdlc/skills/define/reference/<level>-guide.md` to understand required fields and draft template
```
To:
```
2. Read the template for the level at `${CLAUDE_PLUGIN_ROOT}/templates/<level>-template.md` to understand required fields and draft structure
```

- [ ] **Step 3: Add size validation to "What to Check" table**

Add a row to the "What to Check" table:
```
| Size Validation | For feature drafts: `size` field present in frontmatter (must be `small` or `large`). If `size:small`, no `## Stories` section exists. If `size:large`, `## Stories` section exists with at least one item. |
```

- [ ] **Step 4: Commit**

```bash
git add .claude/plugins/sdlc/agents/draft-reviewer/AGENT.md
git commit -m "fix(sdlc): update draft-reviewer template path and add size validation

Template lookup now uses shared templates/ directory instead of
define's reference guides. Added size field validation for features."
```

---

### Task 6: Update Feature Execution Reference

Add `size` field to required fields, make Stories section conditional, add `size:*` label to issue creation.

**Files:**
- Modify: `skills/create/reference/feature-execution.md`

- [ ] **Step 1: Read current feature-execution.md**

```
Read .claude/plugins/sdlc/skills/create/reference/feature-execution.md
```

- [ ] **Step 2: Add size to Required Fields frontmatter**

Add after `parent-epic` line:
```
- `size` — one of: `small`, `large`
```

- [ ] **Step 3: Update Required Fields body sections**

Change:
```
- `## Stories` — at least one checklist item with `(#TBD)` placeholder
```
To:
```
- `## Stories` — **only if `size: large`**: at least one checklist item with `(#TBD)` placeholder. **Must NOT exist if `size: small`.**
```

- [ ] **Step 4: Add size label to issue creation command**

In Step 1 (Create the Feature Issue), add `--label "size:<size>"` to the `gh issue create` command:
```bash
FEAT_URL=$(gh issue create \
  --title "<name>" \
  --body-file /tmp/sdlc-feature-body.md \
  --label "type:feature" \
  --label "priority:<priority>" \
  --label "size:<size>" \
  --label "area:<area1>" \
  --label "area:<area2>")
```

- [ ] **Step 5: Make Step 2 (Create Stub Story Issues) conditional**

Wrap the story stub creation section with:
```
### 2. Create Stub Story Issues (size:large only)

**Skip this step entirely if `size: small`.** Size:small features have no child stories.

For each item in the `## Stories` checklist...
```

- [ ] **Step 6: Update Report Format**

Add size label to the created feature line:
```
>   - Labels: `type:feature`, `priority:<priority>`, `size:<size>`, `area:<areas>`
```

- [ ] **Step 7: Commit**

```bash
git add .claude/plugins/sdlc/skills/create/reference/feature-execution.md
git commit -m "feat(sdlc): add size field support to feature execution reference

Size is now a required frontmatter field. Stories section is conditional
on size:large. size:small features skip story stub creation entirely."
```

---

### Task 7: Update Feature Update Reference

Add size label change examples and escalation rules.

**Files:**
- Modify: `skills/update/reference/feature-update.md`

- [ ] **Step 1: Read current feature-update.md**

```
Read .claude/plugins/sdlc/skills/update/reference/feature-update.md
```

- [ ] **Step 2: Add size label change examples**

In the Label Changes section, add an example:
```markdown
### Size label change

```bash
# Change from small to large (or vice versa)
gh issue edit <N> --add-label "size:large" --remove-label "size:small"
```

**Note:** Changing size from `small` to `large` means the feature now needs stories. Changing from `large` to `small` means existing child stories should be reviewed. If this size change is combined with adding/removing stories, ESCALATE to define — that's a scope change requiring brainstorming.
```

- [ ] **Step 3: Commit**

```bash
git add .claude/plugins/sdlc/skills/update/reference/feature-update.md
git commit -m "feat(sdlc): add size label change handling to feature update reference"
```

---

### Task 8: Update Init Skill

Add `size:small` and `size:large` label creation.

**Files:**
- Modify: `skills/init/SKILL.md`

- [ ] **Step 1: Read current init SKILL.md**

```
Read .claude/plugins/sdlc/skills/init/SKILL.md
```

- [ ] **Step 2: Add size labels to Step 3**

After the triage label creation and before the area labels loop, add:
```bash
# Size labels (orange)
gh label create "size:small" --color "e4e669" --force
gh label create "size:large" --color "e4e669" --force
```

- [ ] **Step 3: Update label count in Step 7**

Change "Universal: 12 labels" to "Universal: 14 labels (type, status, priority, size, triage)"

- [ ] **Step 4: Commit**

```bash
git add .claude/plugins/sdlc/skills/init/SKILL.md
git commit -m "feat(sdlc): add size:small and size:large labels to init skill"
```

---

### Task 9: Update Reconcile Skill

Add size label validation check for features.

**Files:**
- Modify: `skills/reconcile/SKILL.md`

- [ ] **Step 1: Read current reconcile SKILL.md**

```
Read .claude/plugins/sdlc/skills/reconcile/SKILL.md
```

- [ ] **Step 2: Update process flow diagram**

Change `"Run 7 checks" [shape=box]` to `"Run 8 checks" [shape=box]` in the dot diagram.

- [ ] **Step 3: Add size label validation check**

Add a new check to the reconcile checklist (after the existing checks):

```markdown
### Check 8: Size Label Validation

For every issue with `type:feature` label:

1. **Exactly one size label** — must have either `size:small` or `size:large` (not both, not neither)
2. **size:small consistency** — should have zero child stories (issues referencing this feature as parent)
3. **size:large consistency** — should have at least one child story
4. **Non-features with size labels** — any issue without `type:feature` that has `size:small` or `size:large` → strip the size label

**Fix commands:**
- Missing size label on feature: `gh issue edit <N> --add-label "size:small"` (default to small, flag for review)
- Both size labels: `gh issue edit <N> --remove-label "size:small"` (keep large if it has children, keep small if not)
- Size label on non-feature: `gh issue edit <N> --remove-label "size:small"` or `--remove-label "size:large"`
```

- [ ] **Step 4: Commit**

```bash
git add .claude/plugins/sdlc/skills/reconcile/SKILL.md
git commit -m "feat(sdlc): add size label validation to reconcile skill"
```

---

### Task 10: Update PRD

Update artifact definitions, label taxonomy, decision log, and acceptance criteria to reflect the new model.

**Files:**
- Modify: `.claude/sdlc/prd/PRD.md`

- [ ] **Step 1: Read current PRD**

```
Read .claude/sdlc/prd/PRD.md
```

- [ ] **Step 2: Update Architecture — Key architectural decisions**

Replace the depth-based discovery bullet:
```
- Depth-based discovery — artifact complexity assessed via objective criteria (file count, area span, novelty) determines question depth (LIGHT/STANDARD/DEEP)
```
With:
```
- Creative brainstorming with lightweight checklists — define skill uses free-form conversation guided by internal checklists, replacing the rigid depth-based question system
- Amended two-phase principle — define may dispatch operational agents for side-effect artifacts (PI updates, parent issue updates) confirmed by the user during impact analysis; primary artifact still follows define → create/update
```

- [ ] **Step 3: Update Data Models — Artifact Types table**

Update the Feature row's Key Fields:
```
| Feature | GitHub Issue (`type:feature`) | title, Description, size (small/large), Acceptance Criteria, Stories checklist (if size:large), Non-goals, Dependencies, Parent Epic link |
```

Update the PI row's Key Fields:
```
| PI | `.claude/sdlc/pi/PI.md` (git, archived to `completed/PI-N.md`) | name, theme, started, target, Goals, Epics (with scope seeds), Dependencies, Worktree Strategy |
```

- [ ] **Step 4: Update Label Taxonomy table**

Add a Size row:
```
| Size | `size:small`, `size:large` | Classify feature complexity — small features are directly implementable, large features decompose into stories |
```

- [ ] **Step 5: Update Acceptance Criteria**

Update the define bullet:
```
- [ ] `sdlc:define` supports creative brainstorming with level-optional invocation, scope classification, and impact analysis with agent dispatch for each artifact level
```

Add new acceptance criteria:
```
- [ ] `size:small` features can be created without a Stories section
- [ ] `size:large` features require at least one story
- [ ] Impact analysis correctly identifies cascading updates to PI, parent issues, and PRD
- [ ] Operational agents (create-agent, update-agent) successfully execute dispatched work
```

- [ ] **Step 6: Update Decision Log**

Add new entries:
```
| 2026-03-20 | Creative brainstorming replaces depth system | Depth-based discovery (LIGHT/STANDARD/DEEP) made brainstorming feel rigid and formulaic; creative freedom with lightweight checklists produces better artifacts | Define, Reference Guides |
| 2026-03-20 | Flexible artifact hierarchy (optional stories) | Features don't always need stories — small features are directly implementable, simplifying the workflow | Define, Create, Feature Template |
| 2026-03-20 | Amended two-phase principle with impact analysis | Define needs to dispatch operational agents for side-effect artifacts (PI updates, parent updates) during impact analysis; primary artifact still follows define → create | Define, Create, Update, Agents |
| 2026-03-20 | Size labels for features (size:small, size:large) | Visible classification of feature complexity enables better planning, validation, and reconciliation | Labels, Init, Reconcile, Feature Execution |
```

- [ ] **Step 7: Update Roadmap**

Remove or mark as complete the items addressed by this restructure. Update item 5:
```
5. ~~Agents~~ — ✅ Three operational agents (impact-analysis, create-agent, update-agent) dispatched by define during impact analysis
```

- [ ] **Step 8: Commit**

```bash
git add .claude/sdlc/prd/PRD.md
git commit -m "docs(prd): update artifact model, label taxonomy, and decision log

Reflect new flexible hierarchy (optional stories), size labels,
creative brainstorming replacing depth system, amended two-phase
principle, and operational agent dispatch."
```

---

### Task 11: Update CLAUDE.md

Update project conventions and structure to reflect the restructure.

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Read current CLAUDE.md**

```
Read CLAUDE.md
```

- [ ] **Step 2: Update Project Structure**

Update the plugin structure section to include templates and agents:
```
.claude/plugins/sdlc/          # Plugin source
  plugin.json                  # Manifest
  templates/                   # Shared draft templates (rigid output formats)
  skills/                      # 8 skills
    init/ capture/ define/ create/ update/ status/ reconcile/ retro/
  agents/                      # 4 agents
    draft-reviewer/ create-agent/ update-agent/ impact-analysis-agent/
```

- [ ] **Step 3: Update SDLC Workflow**

Update the define line:
```
2. `/sdlc:define [level]` — brainstorm and produce a local draft (level is optional — skill classifies if not specified)
```

- [ ] **Step 4: Update Conventions**

Update hierarchy:
```
- Artifact hierarchy: PRD > PI > Epic > Feature > optional Story
- Features are classified as size:small (directly implementable) or size:large (decomposed into stories)
```

Update two-phase:
```
- Two-phase workflow: define (brainstorm) -> create/update (execute). Define may dispatch agents for side-effect updates confirmed by the user during impact analysis.
```

- [ ] **Step 5: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md for plugin restructure

Reflect templates directory, agents, optional stories, size labels,
level-optional define invocation, and amended two-phase principle."
```

---

### Task 12: Remove Flat Epic References

The new model requires Epic > Feature > optional Story. Flat epics (stories directly under an epic, no features) are removed. Several execution references and reconcile have flat epic code paths that must be cleaned up.

**Files:**
- Modify: `skills/create/reference/epic-execution.md`
- Modify: `skills/create/reference/story-execution.md`
- Modify: `skills/update/reference/story-update.md`
- Modify: `skills/reconcile/SKILL.md`

- [ ] **Step 1: Update epic-execution.md — Required Fields**

Change the body sections requirement:
```
- `## Features` — at least one checklist item with `(#TBD)` placeholder
```
Remove the `or ## Stories` alternative and flat epic references.

- [ ] **Step 2: Update epic-execution.md — Remove story stubs path**

In Step 2 (Create Stub Child Issues), remove the entire "Story stubs (flat epic)" code block and the conditional branching. Only the "Feature stubs" path remains. Update the section header from "Create Stub Child Issues" to "Create Stub Feature Issues".

Remove the `- Feature: none (flat epic)` pattern from the story stub body.

- [ ] **Step 3: Update story-execution.md — Required Fields**

Change `parent-feature` from:
```
- `parent-feature` — issue number of the parent feature (or `none` for flat epics)
```
To:
```
- `parent-feature` — issue number of the parent feature
```

- [ ] **Step 4: Update story-execution.md — Remove flat epic path in Step 3**

In Step 3 (Update Parent Feature's Stories Checklist), remove the "For flat epics" alternate code path. Stories always have a parent feature now.

- [ ] **Step 5: Update story-update.md — Remove flat epic references**

Read `skills/update/reference/story-update.md` and remove any references to "flat epics", `parent-feature: none`, or `Feature: none (flat epic)` patterns.

- [ ] **Step 6: Update reconcile SKILL.md — Remove flat epic exception**

In the C2 Broken Hierarchy check, remove the exception:
```
If a story's ## Parent section contains `- Feature: none` or `- Feature: none (flat epic)`, this is valid
```
Stories must always have a valid parent feature now.

- [ ] **Step 7: Commit**

```bash
git add .claude/plugins/sdlc/skills/create/reference/epic-execution.md \
       .claude/plugins/sdlc/skills/create/reference/story-execution.md \
       .claude/plugins/sdlc/skills/update/reference/story-update.md \
       .claude/plugins/sdlc/skills/reconcile/SKILL.md
git commit -m "feat(sdlc): remove flat epic support from execution references

Flat epics (stories directly under an epic) are replaced by the
flexible hierarchy: Epic > Feature (size:small or size:large) >
optional Story. All execution references and reconcile updated."
```

---

### Task 13: Migrate Existing PI (if exists)

Check if an active PI exists and migrate it to the lighter scope-seeds format.

**Files:**
- Modify: `.claude/sdlc/pi/PI.md` (if it exists)

- [ ] **Step 1: Check if PI exists**

```bash
test -f .claude/sdlc/pi/PI.md && echo "PI exists" || echo "No active PI"
```

If no active PI exists, skip this task entirely.

- [ ] **Step 2: Read current PI**

```
Read .claude/sdlc/pi/PI.md
```

- [ ] **Step 3: Migrate to scope-seeds format**

Rewrite the PI to use the new template format from `templates/pi-template.md`:
- Replace detailed feature lists under epics with scope seeds (bullet points)
- Remove `#TBD` placeholders for features (scope seeds don't have issue numbers)
- Keep existing epic issue numbers if they exist
- Simplify the Dependencies section to cross-epic only
- Update frontmatter if needed

- [ ] **Step 4: Commit**

```bash
git add .claude/sdlc/pi/PI.md
git commit -m "docs(pi): migrate to scope-seeds format

Replace detailed feature lists with lightweight scope seeds.
Epics are now goals with bullet-point direction, not feature commitments."
```

---

### Task 14: Bump Plugin Version and Align Versions

**Files:**
- Modify: `plugin.json`

- [ ] **Step 1: Update version**

Change version from `"0.1.0"` to `"0.2.0"` in `.claude/plugins/sdlc/plugin.json`.

- [ ] **Step 2: Commit**

```bash
git add .claude/plugins/sdlc/plugin.json
git commit -m "chore(sdlc): bump plugin version to 0.2.0"
```

---

## Task Dependencies

```
Task 1 (Templates) ──┐
                      ├── Task 3 (Rewrite Define) ── Task 4 (New Agents)
Task 2 (Brainstorm) ──┘
                                                      │
Task 5 (Draft Reviewer) ← depends on Task 1           │
Task 6 (Feature Execution) ← depends on Task 1        │
Task 7 (Feature Update) ← independent                 │
Task 8 (Init) ← independent                           │
Task 9 (Reconcile) ← independent                      │
Task 12 (Flat Epic Removal) ← independent             │
                                                      │
Task 10 (PRD) ← depends on Tasks 1-12 being designed  │
Task 11 (CLAUDE.md) ← depends on Tasks 1-12           │
Task 13 (PI Migration) ← depends on Task 1 (templates)│
Task 14 (Version Bump) ← last                         │
```

**Parallelizable groups:**
- Tasks 1 + 2 can run in parallel (no dependencies)
- Tasks 5 + 6 + 7 + 8 + 9 + 12 can run in parallel (independent modifications)
- Tasks 10 + 11 + 13 can run in parallel (independent files)
- Task 14 runs last
