# Define Skill Overhaul Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure sdlc:define to use native Claude Code patterns (TaskCreate, Agent dispatch, one-task-at-a-time) while preserving the LIGHT/STANDARD/DEEP depth system via enriched reference guides.

**Architecture:** Three components — a concise SKILL.md orchestrator (~150 lines) with hardcoded 9-task checklist, 5 enriched reference guides with depth-specific branching, and a draft-reviewer agent for automated quality gates.

**Tech Stack:** Claude Code plugin system (markdown skills, YAML frontmatter, Agent dispatch)

**Spec:** `docs/specs/2026-03-20-define-skill-overhaul-design.md`

---

### Task 1: Create the draft-reviewer agent

**Files:**
- Create: `.claude/plugins/sdlc/agents/draft-reviewer/AGENT.md`

- [ ] **Step 1: Create the agents directory**

```bash
mkdir -p .claude/plugins/sdlc/agents/draft-reviewer
```

- [ ] **Step 2: Write the AGENT.md file**

Create `.claude/plugins/sdlc/agents/draft-reviewer/AGENT.md` with the following content:

```markdown
---
name: draft-reviewer
description: Reviews SDLC draft artifacts for completeness, consistency with upstream artifacts, and readiness for sdlc:create execution. Use when Task 7 of sdlc:define dispatches you.
tools: Read, Bash, Grep, Glob
---

You are a draft reviewer for the SDLC plugin. You review draft artifacts before they are presented to the user.

## Input

You receive:
- **Draft file path** — the draft to review
- **Level** — prd, pi, epic, feature, or story
- **Upstream artifact paths** — files and GitHub issue numbers to validate against

## What to Check

| Category | What to Look For |
|----------|------------------|
| Completeness | Empty sections, `[TBD]` placeholders, missing required fields per the level's draft template |
| Upstream Consistency | Draft contradicts PRD constraints, PI scope, parent epic/feature body |
| Internal Consistency | Draft sections contradict each other (e.g., success criteria don't match overview) |
| Dependency Validity | Referenced issue numbers exist and are the correct type (verify via `gh issue view <number> --json title,labels,state`) |
| Scope | Draft scope matches the level (not a story masquerading as an epic, features match parent epic) |
| YAGNI | Unrequested sections, over-specification for the level |

## What NOT to Check

- Stylistic preferences or wording quality
- "Could be more detailed" suggestions
- Creative decisions (that's the user's job)
- Anything that's opinion rather than verifiable fact

## Calibration

Only flag issues that would cause real problems when `sdlc:create` or `sdlc:update` tries to execute this draft. A missing required field, a contradiction with the PRD, or a dependency on a nonexistent issue — those are issues. Minor wording improvements and stylistic preferences are not. Approve unless there are serious gaps.

## Process

1. Read the draft file
2. Read the reference guide for the level at `.claude/plugins/sdlc/skills/define/reference/<level>-guide.md` to understand required fields and draft template
3. Read each upstream artifact specified in the dispatch context
4. For any GitHub issue references (`#N`), verify they exist: `gh issue view <N> --json title,labels,state`
5. Compare draft against upstream artifacts for consistency
6. Produce your review

## Output Format

```
Status: [Approved | Issues Found]

Issues:
1. [Section]: [specific problem and why it matters for sdlc:create execution]
2. [Section]: [specific problem and why it matters]

Recommendations (advisory, not blocking):
- [suggestion]
```

If no issues found, output:
```
Status: Approved

No issues found. Draft is ready for user review.
```
```

- [ ] **Step 3: Verify the file is well-formed**

```bash
head -5 .claude/plugins/sdlc/agents/draft-reviewer/AGENT.md
```

Expected: YAML frontmatter with `---`, name, description, tools fields.

- [ ] **Step 4: Commit**

```bash
git add .claude/plugins/sdlc/agents/draft-reviewer/AGENT.md
git commit -m "feat(sdlc): add draft-reviewer agent for automated draft quality gates"
```

---

### Task 2: Rewrite SKILL.md

**Files:**
- Modify: `.claude/plugins/sdlc/skills/define/SKILL.md` (complete rewrite)

- [ ] **Step 1: Write the new SKILL.md**

Replace the entire content of `.claude/plugins/sdlc/skills/define/SKILL.md` with:

```markdown
---
name: define
description: Use when defining new SDLC artifacts (PRD, PI, epic, feature, story) or reshaping existing ones through collaborative brainstorming that produces a reviewable local draft.
allowed-tools: Read, Edit, Write, Bash, Grep, Glob, Agent
argument-hint: "[level] [identifier]"
---

I'm using the sdlc:define skill to define/reshape an SDLC artifact.

<HARD-GATE>
Do NOT produce a draft without completing all tasks in order. Do NOT skip a task because the artifact "seems simple." Do NOT present multiple tasks at once — complete each one before moving to the next.
</HARD-GATE>

## Red Flags

| Thought | Reality |
|---------|---------|
| "This is simple, skip to the draft" | All tasks are mandatory. Simple artifacts go through LIGHT depth, not skipped tasks. |
| "I already know what to build" | Discovery catches assumptions. Even if you're right, the user confirms. |
| "The reference guide isn't needed" | The guide has scope criteria, question templates, and draft template. Always load it. |
| "Let me skip the approaches phase" | LIGHT gets a single approach confirmation. It takes 10 seconds. Don't skip it. |
| "The draft is obvious, no review needed" | The draft reviewer catches issues you miss after a long conversation. Always run it. |
| "I'll present multiple tasks at once" | One task at a time. Complete each before moving to the next. |
| "I'll assess scope from intuition" | Scope criteria are objective in the reference guide. Use those signals, not vibes. |
| "The reviewer will catch everything" | The reviewer catches structural issues. Creative decisions are the user's job in Task 8. |

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Load context** — Parse level from `$ARGUMENTS`. Load `${CLAUDE_PLUGIN_ROOT}/skills/define/reference/<level>-guide.md`. Read upstream artifacts per guide's context table and pre-flight checklist. Detect new vs reshape: issue number in args = reshape; "reshape/rethink/revise/update" = reshape; existing draft found = ask user. For reshapes, load current state from GitHub issue or git file.
2. **Assess scope** — Evaluate depth (LIGHT/STANDARD/DEEP) using guide's scope assessment criteria matrix. Announce result with specific signals: "This looks like a STANDARD scope — [concrete reasons from the criteria]."
3. **Run discovery** — Follow guide's "Discovery by Depth" section. Ask questions ONE AT A TIME. Wait for user's answer before asking the next. Dispatch research Agent when guide instructs (STANDARD/DEEP).
4. **Re-evaluate scope** — Post-discovery gate. Re-assess depth using same criteria from guide. Depth can only go UP (max one upgrade per session). If upgraded, re-run discovery at new depth — ask only the additional questions, not ones already answered.
5. **Present approaches** — Follow guide's "Approaches by Depth" section. LIGHT: 1 approach, confirm. STANDARD: 2 options with trade-offs, recommend one. DEEP: 3 with trade-off table. Wait for user's choice.
6. **Write draft** — Use guide's draft body template. Write to `.claude/sdlc/drafts/<level>-<name>.md` (new) or `.claude/sdlc/drafts/<level>-<issue-number>.md` (reshape). Reshapes MUST include a `## Changes` section documenting what was modified. Ensure `.claude/sdlc/drafts/` directory exists.
7. **Draft review loop** — Dispatch the `draft-reviewer` agent with: draft file path, level, and upstream artifact paths from the guide's "Review Context" section. Fix any issues the agent finds in the draft. Re-dispatch (max 3 iterations). If still failing after 3, surface remaining issues to the user.
8. **User reviews draft** — Present the full draft to the user. Do not summarize — show everything. Ask: "Want to change anything?" Loop until explicit approval. Do NOT interpret silence or ambiguity as approval.
9. **Announce next step** — For new artifacts: "Draft saved to `<path>`. Run `/sdlc:create <level>` when ready to push it live." For reshapes: "Draft saved with changes documented. Run `/sdlc:update <level> <number>` to apply the changes." Do NOT auto-invoke create or update.

## Dependency Format

All dependency references in drafts use this exact format:

- Blocked by: #N, #M
- Blocks: #N, #M

Rules: dash-prefixed, `#` prefix on issue numbers, comma-space separated, `none` when empty. Bidirectional — reconcile enforces consistency.

## Reshape Drafts: Changes Section

All reshapes MUST include this section at the end of the draft:

| Section | Change | Old Value | New Value |
|---------|--------|-----------|-----------|

This section is consumed by `sdlc:update` to apply surgical edits.

## Integration

| Scenario | Flow |
|----------|------|
| New artifact | `sdlc:define` produces draft → `sdlc:create` pushes to GitHub/git |
| Reshape existing | `sdlc:define` produces draft with Changes section → `sdlc:update` applies edits |
```

- [ ] **Step 2: Verify line count and structure**

```bash
wc -l .claude/plugins/sdlc/skills/define/SKILL.md
```

Expected: ~100-150 lines (significantly shorter than the previous ~390 lines).

```bash
head -6 .claude/plugins/sdlc/skills/define/SKILL.md
```

Expected: YAML frontmatter with name, description, allowed-tools, argument-hint.

- [ ] **Step 3: Commit**

```bash
git add .claude/plugins/sdlc/skills/define/SKILL.md
git commit -m "feat(sdlc): rewrite define SKILL.md as concise orchestrator with task tracking"
```

---

### Task 3: Enrich prd-guide.md

**Files:**
- Modify: `.claude/plugins/sdlc/skills/define/reference/prd-guide.md`

- [ ] **Step 1: Add four new sections to the end of prd-guide.md**

Append the following sections after the existing `## Draft Body Template` section:

```markdown

## Pre-Flight Checklist

- Check if `.claude/sdlc/prd/PRD.md` exists:
  - **No PRD + no codebase** = GREENFIELD (building from nothing)
  - **No PRD + codebase exists** = BROWNFIELD (documenting what already exists)
  - **PRD exists** = RESHAPE (modifying an existing PRD)
- For brownfield: scan `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, directory structure for existing tech stack

## Discovery by Depth

### LIGHT (reshape — minor section update)
Summarize what's changing. Ask at most 1 confirming question. Use reshape questions from templates above.

### STANDARD (reshape — new section or major revision)
Ask 2-4 targeted questions from reshape templates above, one at a time.

### DEEP (greenfield or brownfield — full PRD from scratch)

**Greenfield:** Full 11-question interview from templates above, one at a time. The user is the sole source of information.

**Brownfield:** Before asking questions, dispatch a research Agent:

Prompt: "Analyze this repository and report: 1. Tech stack (languages, frameworks, package managers — read package.json, pyproject.toml, etc.) 2. Directory structure (top-level layout, key directories) 3. Architecture patterns (monolith vs services, API framework, database ORM) 4. Data models (entity names, schemas, relationships) 5. API endpoints (routes, auth patterns) 6. Security patterns (auth mechanism, middleware, environment usage). Do not modify any files."

Then present findings to the user and ask 4-6 targeted gap questions from brownfield templates above.

## Approaches by Depth

### LIGHT
Present one approach for the section update. Confirm with user.

### STANDARD
Present two options for the revision. Recommend one with rationale.

### DEEP
Present three approaches with trade-offs (e.g., for architecture decisions, tech stack choices). Use a trade-off table.

## Review Context

The draft reviewer should check this draft against:
- Codebase scan findings (brownfield) for tech stack accuracy
- User's stated requirements from discovery for completeness
- Internal consistency across all PRD sections (e.g., tech stack matches architecture, data models match API contracts)
```

- [ ] **Step 2: Verify the file has the new sections**

```bash
grep "^## " .claude/plugins/sdlc/skills/define/reference/prd-guide.md
```

Expected: Should show the original sections plus Pre-Flight Checklist, Discovery by Depth, Approaches by Depth, Review Context.

- [ ] **Step 3: Commit**

```bash
git add .claude/plugins/sdlc/skills/define/reference/prd-guide.md
git commit -m "feat(sdlc): enrich prd-guide with depth branching and review context"
```

---

### Task 4: Enrich pi-guide.md

**Files:**
- Modify: `.claude/plugins/sdlc/skills/define/reference/pi-guide.md`

- [ ] **Step 1: Add four new sections to the end of pi-guide.md**

Append after `## Draft Body Template`:

```markdown

## Pre-Flight Checklist

- PRD exists at `.claude/sdlc/prd/PRD.md`
- Check for previous PI retros in `.claude/sdlc/retros/` (carry-over items affect discovery)
- Check if active PI exists at `.claude/sdlc/pi/PI.md` (if yes, sdlc:create will archive it)

## Discovery by Depth

### LIGHT (1-2 epics, no carry-over, no complex dependencies)
Present the PRD roadmap and summarize what you'd include. Ask 2-3 confirming questions from templates above (roadmap selection, theme, target date).

If carry-over items exist from a previous retro, ask about them first.

### STANDARD (3-4 epics, some dependencies, or carry-over items)
Ask 4-5 questions from templates above, one at a time: roadmap selection, theme, target date, epic breakdown, dependencies.

If carry-over items exist, add the carry-over question before roadmap selection.

### DEEP (5+ epics, complex dependencies, significant re-planning)
Ask all 6 questions from templates above plus follow-ups on dependency ordering and worktree coordination.

If carry-over items exist, start with carry-over assessment.

## Approaches by Depth

### LIGHT
Present one approach for the PI structure. Confirm with user.

### STANDARD
Present two options for epic grouping or sequencing. Recommend one.

### DEEP
Present three approaches for the PI structure (e.g., different epic groupings, different parallelism strategies). Use a trade-off table.

## Review Context

The draft reviewer should check this draft against:
- `.claude/sdlc/prd/PRD.md` — roadmap alignment (are PI epics drawn from the roadmap?)
- Previous retro at `.claude/sdlc/retros/` — carry-over items addressed
- Internal consistency (dependency graph matches epic/feature listings, worktree strategy is feasible)
```

- [ ] **Step 2: Verify the file has the new sections**

```bash
grep "^## " .claude/plugins/sdlc/skills/define/reference/pi-guide.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/plugins/sdlc/skills/define/reference/pi-guide.md
git commit -m "feat(sdlc): enrich pi-guide with depth branching and review context"
```

---

### Task 5: Enrich epic-guide.md

**Files:**
- Modify: `.claude/plugins/sdlc/skills/define/reference/epic-guide.md`

- [ ] **Step 1: Add four new sections to the end of epic-guide.md**

Append after `## Draft Body Template`:

```markdown

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
```

- [ ] **Step 2: Verify the file has the new sections**

```bash
grep "^## " .claude/plugins/sdlc/skills/define/reference/epic-guide.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/plugins/sdlc/skills/define/reference/epic-guide.md
git commit -m "feat(sdlc): enrich epic-guide with depth branching and review context"
```

---

### Task 6: Enrich feature-guide.md

**Files:**
- Modify: `.claude/plugins/sdlc/skills/define/reference/feature-guide.md`

- [ ] **Step 1: Add four new sections to the end of feature-guide.md**

Append after `## Draft Body Template`:

```markdown

## Pre-Flight Checklist

- PRD exists at `.claude/sdlc/prd/PRD.md`
- PI exists at `.claude/sdlc/pi/PI.md`
- Parent epic issue is resolvable via `gh issue view <epic-number> --json title,body,labels`

## Discovery by Depth

### LIGHT (1-2 stories, clear pattern, no cross-feature deps)
Summarize from parent epic context. Ask 1 confirming question. Use question 1 (description) from templates above and infer the rest from the epic body.

### STANDARD (3-4 stories, extends patterns)
Ask questions 1-3 from templates above, one at a time. Infer dependencies from parent epic and PI context.

### DEEP (5+ stories, new patterns, cross-feature deps)
Ask all 4 questions from templates above plus follow-ups on story breakdown and architecture implications.

## Approaches by Depth

### LIGHT
Present one approach for the feature's story breakdown. Confirm with user.

### STANDARD
Present two options for how to decompose into stories (e.g., different groupings or sequencing). Recommend one.

### DEEP
Present three approaches with trade-off table.

## Review Context

The draft reviewer should check this draft against:
- `.claude/sdlc/prd/PRD.md` — architecture, security constraints
- `.claude/sdlc/pi/PI.md` — feature alignment with PI scope
- Parent epic via `gh issue view #<epic-number>` — feature fits within epic's scope and success criteria, stories match epic's feature listing
```

- [ ] **Step 2: Verify the file has the new sections**

```bash
grep "^## " .claude/plugins/sdlc/skills/define/reference/feature-guide.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/plugins/sdlc/skills/define/reference/feature-guide.md
git commit -m "feat(sdlc): enrich feature-guide with depth branching and review context"
```

---

### Task 7: Enrich story-guide.md

**Files:**
- Modify: `.claude/plugins/sdlc/skills/define/reference/story-guide.md`

- [ ] **Step 1: Add four new sections to the end of story-guide.md**

Append after `## Draft Body Template`:

```markdown

## Pre-Flight Checklist

- PRD exists at `.claude/sdlc/prd/PRD.md`
- Parent feature issue is resolvable via `gh issue view <feature-number> --json title,body,labels`
- Parent epic issue is resolvable via `gh issue view <epic-number> --json title,body,labels`

## Discovery by Depth

### LIGHT (1 file, clear pattern, 0 blockers, detailed parent context)
Summarize from parent feature context. Confirm with 1 question. Use question 1 (description) from templates above, infer acceptance criteria and file scope from parent context and codebase patterns.

### STANDARD (2-3 files, extends patterns, 1-2 blockers)
Dispatch a research Agent before asking questions:

Prompt: "Search for existing patterns related to [story topic]: 1. Find files that will be modified (exact paths) 2. Identify existing patterns to follow (similar implementations) 3. Check for related tests (test file locations, test patterns used) 4. Note any utilities or helpers that should be reused. Do not modify any files."

Then ask questions 1-3 from templates above, one at a time. Infer technical notes and dependencies from Agent findings and parent context.

### DEEP (4+ files, new patterns, 3+ blockers, sparse parent context)
Dispatch research Agent first (same prompt as STANDARD). Then ask all 5 questions from templates above, plus follow-ups on edge cases, error handling, and testing strategy.

## Approaches by Depth

### LIGHT
Present one implementation approach. Confirm with user.

### STANDARD
Present two options (e.g., different implementation strategies or patterns to follow). Recommend one.

### DEEP
Present three approaches with trade-off table (e.g., different architectural patterns, different testing strategies).

## Review Context

The draft reviewer should check this draft against:
- `.claude/sdlc/prd/PRD.md` — security constraints (auth, data sensitivity), data models, API contracts, tech stack
- Parent feature via `gh issue view #<feature-number>` — story fits within feature scope
- Parent epic via `gh issue view #<epic-number>` — story aligns with epic's success criteria
- Acceptance criteria are testable, specific, and complete (not vague — per the AC guidance in this guide)
```

- [ ] **Step 2: Verify the file has the new sections**

```bash
grep "^## " .claude/plugins/sdlc/skills/define/reference/story-guide.md
```

- [ ] **Step 3: Commit**

```bash
git add .claude/plugins/sdlc/skills/define/reference/story-guide.md
git commit -m "feat(sdlc): enrich story-guide with depth branching and review context"
```
