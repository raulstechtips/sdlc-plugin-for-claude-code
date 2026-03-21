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
- `skills/update/reference/feature-update.md` — Add size label change examples
- `agents/draft-reviewer/AGENT.md` — Update template lookup path, add size validation
- `skills/init/SKILL.md` — Add `size:small` and `size:large` label creation
- `skills/reconcile/SKILL.md` — Add size label validation check
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

The new skill must include:
- Frontmatter: `argument-hint: "[level] [identifier]"` (level is optional now)
- Announcement message
- Hard gate (no draft without completing all phases, no skipping)
- Anti-rationalization table (updated — remove depth references, add impact analysis references)
- 9-step process flow:
  1. Load context — Read PRD, PI, existing artifacts. Parse optional level from `$ARGUMENTS`. If level provided, load brainstorming guide. Pre-flight checks per level.
  2. Creative brainstorming — Load brainstorming guide from `${CLAUDE_PLUGIN_ROOT}/skills/define/reference/<level>-brainstorm.md`. If no level yet, start general conversation. Ask follow-ups naturally, check off internal checklist items as conversation covers them. One question at a time, wait for answers.
  3. Scope classification — If level not specified, recommend with reasoning. Can happen naturally mid-brainstorm. Reclassification allowed without restarting.
  4. Propose approaches — 2-3 ways to structure/decompose the work. Trade-offs and recommendation. Reclassification can happen here too.
  5. Draft — Load template from `${CLAUDE_PLUGIN_ROOT}/templates/<level>-template.md`. Pour brainstorm content into rigid format. Write to `.claude/sdlc/drafts/<level>-<name>.md`. Reshapes include `## Changes` section.
  6. Draft review loop — Dispatch `draft-reviewer` agent with draft path, level, upstream paths. Fix issues, re-dispatch (max 3). Surface to user if still failing.
  7. User reviews draft — Present full draft. "Want to change anything?" Loop until explicit approval.
  8. Impact analysis — Dispatch `impact-analysis-agent` with brainstorm context. Present impacts one by one. Each is a mini-conversation. Dispatch `create-agent` or `update-agent` as each is confirmed.
  9. Announce next step — "Draft saved. Run `/sdlc:create <level>` when ready, or I can dispatch the create agent now." (Primary artifact only — side-effects handled in Step 8.)
- Reshape flow section (detection heuristics, Changes section requirement)
- Pre-flight checks table
- Draft file naming convention
- Dependency format (unchanged)
- Integration table (updated to include agent dispatch path)
- Internal signals for "I have enough to draft"

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
- Frontmatter: `name: create-agent`, `description: Creates SDLC artifacts from drafts — dispatched by define or impact analysis, not user-invoked`, `tools: Read, Bash, Grep, Glob`
- Input contract: draft file path, artifact level
- Behavior: load template from `${CLAUDE_PLUGIN_ROOT}/templates/<level>-template.md`, load execution reference from `${CLAUDE_PLUGIN_ROOT}/skills/create/reference/<level>-execution.md`, follow exactly
- What it skips (vs the skill): no user confirmation, no draft cleanup offer, no cascade logic
- Output: created artifact identifier (issue number + URL, or file path)

- [ ] **Step 4: Write update-agent AGENT.md**

Must include:
- Frontmatter: `name: update-agent`, `description: Updates existing SDLC artifacts — dispatched by define or impact analysis, not user-invoked`, `tools: Read, Bash, Grep, Glob`
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

- [ ] **Step 2: Add size label validation check**

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

- [ ] **Step 3: Commit**

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

### Task 12: Bump Plugin Version

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
                                                      │
Task 10 (PRD) ← depends on Tasks 1-9 being designed   │
Task 11 (CLAUDE.md) ← depends on Tasks 1-9            │
Task 12 (Version Bump) ← last                         │
```

**Parallelizable groups:**
- Tasks 1 + 2 can run in parallel (no dependencies)
- Tasks 5 + 6 + 7 + 8 + 9 can run in parallel (independent modifications)
- Tasks 10 + 11 can run in parallel (independent files)
- Task 12 runs last
