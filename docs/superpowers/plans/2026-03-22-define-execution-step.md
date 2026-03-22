# Define Execution Step & Execution Reference Simplification

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Move artifact creation/update orchestration into define's new Phase 8 (Execution), simplify execution references that no longer create children, and fix a broken file path in update-agent.

**Architecture:** Four independent file edits applied bottom-up — fix the leaf bug first, simplify execution references, then rewrite the orchestrator (SKILL.md). All files are markdown skill/agent definitions, no application code.

**Tech Stack:** Claude Code plugin system (markdown files with YAML frontmatter), `gh` CLI, `git`

---

## File Map

| File | Action | Responsibility |
|------|--------|---------------|
| `.claude/plugins/sdlc/agents/update-agent/AGENT.md` | Modify line 18 | Fix broken path reference |
| `.claude/plugins/sdlc/skills/create/reference/epic-execution.md` | Modify | Remove steps 2-3, renumber, update report |
| `.claude/plugins/sdlc/skills/create/reference/feature-execution.md` | Modify | Remove steps 2-3, renumber, update report |
| `.claude/plugins/sdlc/skills/define/SKILL.md` | Modify | Insert Phase 8, renumber 8→9 and 9→10, rewrite tables |

---

### Task 1: Fix update-agent path reference

**Files:**
- Modify: `.claude/plugins/sdlc/agents/update-agent/AGENT.md:18`

- [ ] **Step 1: Verify the bug exists**

Run: `grep 'execution.md' .claude/plugins/sdlc/agents/update-agent/AGENT.md`
Expected: Line 18 contains `<level>-execution.md`

- [ ] **Step 2: Apply the fix**

Replace line 18:
```
1. Load the execution reference from `${CLAUDE_PLUGIN_ROOT}/skills/update/reference/<level>-execution.md`
```
With:
```
1. Load the update reference from `${CLAUDE_PLUGIN_ROOT}/skills/update/reference/<level>-update.md`
```

- [ ] **Step 3: Verify the fix**

Run: `grep 'update.md\|execution.md' .claude/plugins/sdlc/agents/update-agent/AGENT.md`
Expected: Line 18 now contains `<level>-update.md`, no remaining `execution.md` references

- [ ] **Step 4: Commit**

```bash
git add .claude/plugins/sdlc/agents/update-agent/AGENT.md
git commit -m "fix(sdlc): correct update-agent file path reference from execution.md to update.md"
```

---

### Task 2: Simplify epic-execution.md

**Files:**
- Modify: `.claude/plugins/sdlc/skills/create/reference/epic-execution.md`

- [ ] **Step 1: Verify current structure has 7 steps**

Run: `grep -n '### [0-9]' .claude/plugins/sdlc/skills/create/reference/epic-execution.md`
Expected: Steps 1-7 present

- [ ] **Step 2: Remove steps 2 and 3**

Delete the entire "### 2. Create Stub Feature Issues" section (lines 43-75) and the entire "### 3. Update Epic Issue Body" section (lines 77-93).

- [ ] **Step 3: Renumber remaining steps**

Renumber the surviving steps:
- `### 4. Bidirectional Dependency Linking` → `### 2. Bidirectional Dependency Linking`
- `### 5. Update PI.md` → `### 3. Update PI.md`
- `### 6. Commit PI.md Update` → `### 4. Commit PI.md Update`
- `### 7. Clean Up Temp Files` → `### 5. Clean Up Temp Files`

- [ ] **Step 4: Update Report Format**

Replace the Report Format section with:

```markdown
## Report Format

> **Created:**
> - Epic: #`<EPIC_NUM>` — "`<name>`"
>   - Labels: `type:epic`, `priority:<priority>`, `area:<areas>`
>
> **Updated:**
> - Blocker #`<N>` body: added `Blocks: #<EPIC_NUM>` to Dependencies
> - PI.md: replaced `#TBD` with #`<EPIC_NUM>`
> - Committed: `docs(pi): add issue numbers for epic <name> (#<EPIC_NUM>)`
```

Remove the feature stub lines and the #TBD backfill line — those are now define's Phase 8 responsibility.

- [ ] **Step 5: Verify final structure has 5 steps**

Run: `grep -n '### [0-9]' .claude/plugins/sdlc/skills/create/reference/epic-execution.md`
Expected: Steps 1-5 present, no steps 6-7

- [ ] **Step 6: Commit**

```bash
git add .claude/plugins/sdlc/skills/create/reference/epic-execution.md
git commit -m "refactor(sdlc): remove child-creation steps from epic-execution reference

Define's Phase 8 now orchestrates stub feature creation and #TBD backfill.
Epic execution reference retains only: create epic, dependency linking, PI update, commit, cleanup."
```

---

### Task 3: Simplify feature-execution.md

**Files:**
- Modify: `.claude/plugins/sdlc/skills/create/reference/feature-execution.md`

- [ ] **Step 1: Verify current structure has 7 steps**

Run: `grep -n '### [0-9]' .claude/plugins/sdlc/skills/create/reference/feature-execution.md`
Expected: Steps 1-7 present

- [ ] **Step 2: Remove steps 2 and 3**

Delete the entire "### 2. Create Stub Story Issues (size:large only)" section (lines 42-78) and the entire "### 3. Update Feature Issue Body" section (lines 80-95).

- [ ] **Step 3: Renumber remaining steps**

Renumber the surviving steps:
- `### 4. Update Parent Epic Body` → `### 2. Update Parent Epic Body`
- `### 5. Bidirectional Dependency Linking` → `### 3. Bidirectional Dependency Linking`
- `### 6. Update PI.md (if feature is listed there)` → `### 4. Update PI.md (if feature is listed there)`
- `### 7. Clean Up Temp Files` → `### 5. Clean Up Temp Files`

- [ ] **Step 4: Update Report Format**

Replace the Report Format section with:

```markdown
## Report Format

> **Created:**
> - Feature: #`<FEAT_NUM>` — "`<name>`"
>   - Labels: `type:feature`, `priority:<priority>`, `size:<size>`, `area:<areas>`
>
> **Updated:**
> - Parent epic #`<parent-epic>` body: replaced `#TBD` with #`<FEAT_NUM>`
> - Blocker #`<N>` body: added `Blocks: #<FEAT_NUM>` to Dependencies
> - PI.md: replaced `#TBD` with #`<FEAT_NUM>` *(if applicable)*
> - Committed: `docs(pi): add issue number for feature <name> (#<FEAT_NUM>)` *(if applicable)*
```

Remove the story stub lines and the #TBD backfill line.

- [ ] **Step 5: Verify final structure has 5 steps**

Run: `grep -n '### [0-9]' .claude/plugins/sdlc/skills/create/reference/feature-execution.md`
Expected: Steps 1-5 present, no steps 6-7

- [ ] **Step 6: Commit**

```bash
git add .claude/plugins/sdlc/skills/create/reference/feature-execution.md
git commit -m "refactor(sdlc): remove child-creation steps from feature-execution reference

Define's Phase 8 now orchestrates stub story creation and #TBD backfill.
Feature execution reference retains only: create feature, update parent epic, dependency linking, PI update, cleanup."
```

---

### Task 4: Rewrite SKILL.md — insert Phase 8, renumber, update tables

**Files:**
- Modify: `.claude/plugins/sdlc/skills/define/SKILL.md`

This is the largest task. It has four sub-edits that must be applied in sequence.

- [ ] **Step 1: Verify current structure has 9 phases**

Run: `grep -n '### Phase' .claude/plugins/sdlc/skills/define/SKILL.md`
Expected: Phases 1-9

- [ ] **Step 2: Insert Phase 8 (Execution) after Phase 7**

Insert the following new section after the Phase 7 block (after line 100) and before the current Phase 8:

```markdown
### Phase 8: Execution

Dispatch agents to create or update the primary artifact immediately. The user approved the draft in Phase 7 — execute now, before impact analysis, so downstream phases have real issue numbers.

**Decision tree:**

```
Is this a reshape? (draft has ## Changes section)
├── YES: Is this a reclassification? (draft type ≠ existing issue type label)
│   ├── YES (reclassification reshape):
│   │   1. Dispatch update-agent: swap type label, remove stale labels (e.g., size:* if becoming epic), replace body with draft body
│   │   2. If new level has children (epic/feature-large): dispatch create-agents in parallel for children
│   │   3. If children created: dispatch update-agent to backfill parent body #TBD → real numbers
│   └── NO (normal reshape):
│       1. Dispatch update-agent with Changes table parameters
├── NO (new artifact): What level?
│   ├── PRD/PI:
│   │   1. Dispatch create-agent (handles git add + commit)
│   ├── Epic:
│   │   1. Dispatch create-agent for epic → receive issue number
│   │   2. Dispatch create-agents in parallel for each feature in ## Features checklist (each gets parent-epic = new number)
│   │   3. Collect all feature issue numbers
│   │   4. Dispatch update-agent to backfill epic body: replace each #TBD with real feature number
│   ├── Feature (size:large):
│   │   1. Dispatch create-agent for feature → receive issue number
│   │   2. Dispatch create-agents in parallel for each story in ## Stories checklist (each gets parent-feature = new number, parent-epic from draft frontmatter)
│   │   3. Collect all story issue numbers
│   │   4. Dispatch update-agent to backfill feature body: replace each #TBD with real story number
│   ├── Feature (size:small):
│   │   1. Dispatch create-agent for feature (no children)
│   └── Story:
│       1. Dispatch create-agent for story (no children)
```

**Sequencing rule:** The primary artifact MUST be created first (sequential) because children need its issue number for their `## Parent` section. Children can then be created in parallel. The backfill step MUST wait for all children to complete.

**What define passes to agents:**

To create-agent:
- Draft file path (or inline body for stubs)
- Artifact level
- For child stubs: the child name, one-line description from the parent's checklist, parent issue numbers, inherited priority and area labels

To update-agent:
- Target issue number
- Level
- Change description (from Changes table for reshapes, or "backfill #TBD with #N for child-name" for backfills)
- For reclassification: old type, new type, labels to add, labels to remove, new body content
```

- [ ] **Step 3: Renumber Phase 8 → Phase 9 (Impact Analysis)**

Change `### Phase 8: Impact Analysis` to `### Phase 9: Impact Analysis`. The body of this section is unchanged.

- [ ] **Step 4: Rewrite Phase 9 → Phase 10 (Next Steps)**

Replace the entire current Phase 9 (Announce Next Step) with:

```markdown
### Phase 10: Next Steps

Informational guidance — no dispatching, no "want me to create?". Content is level-dependent:

- **Epic** → "Features #X, #Y, #Z created as stubs. Run `/sdlc:define feature #X` to flesh one out."
- **Feature (large)** → "Stories #A, #B created as stubs. Run `/sdlc:define story #A` to flesh one out."
- **Feature (small)** → "Feature is directly implementable. Ready to develop."
- **Story** → "Next unfinished story under this feature is #B, or all stories defined — ready to develop."
- **PRD/PI** → "Committed. Run `/sdlc:define epic` to start decomposing."
```

- [ ] **Step 5: Update Red Flags table**

Remove the row that references `/sdlc:create`. Currently there is no explicit `/sdlc:create` row, but verify by scanning the table. If no row exists, no change needed.

Additionally, add a new row:

```markdown
| "I'll skip execution, the user can run /sdlc:create" | Phase 8 handles execution. Define is end-to-end now. |
```

- [ ] **Step 6: Update Integration table**

Replace the current Integration table (lines 146-150) with:

```markdown
## Integration

| Scenario | Flow |
|----------|------|
| New artifact | define brainstorms → draft → Phase 8 dispatches create-agent (and create-agents for children if applicable) |
| Reshape existing | define brainstorms → draft with Changes section → Phase 8 dispatches update-agent |
| Side-effect updates | Phase 9 (Impact Analysis) dispatches create-agent/update-agent for confirmed cascading changes |
```

- [ ] **Step 7: Verify final structure has 10 phases**

Run: `grep -n '### Phase' .claude/plugins/sdlc/skills/define/SKILL.md`
Expected: Phases 1-10

- [ ] **Step 8: Commit**

```bash
git add .claude/plugins/sdlc/skills/define/SKILL.md
git commit -m "feat(sdlc): add Phase 8 (Execution) to define skill, renumber phases

Define now orchestrates artifact creation/update immediately after user
approval. Phase 8 dispatches create-agent for primary artifact, parallel
create-agents for children, and update-agent for #TBD backfill.

Impact Analysis (now Phase 9) runs after execution with real issue numbers.
Next Steps (now Phase 10) is informational only — no dispatching.

Red Flags and Integration tables updated to reflect define as end-to-end."
```

---

### Task 5: Final verification

- [ ] **Step 1: Verify all files are consistent**

Run these checks:
```bash
# update-agent references update.md, not execution.md
grep 'execution.md' .claude/plugins/sdlc/agents/update-agent/AGENT.md
# Expected: no output

# epic-execution has 5 steps
grep -c '### [0-9]' .claude/plugins/sdlc/skills/create/reference/epic-execution.md
# Expected: 5

# feature-execution has 5 steps
grep -c '### [0-9]' .claude/plugins/sdlc/skills/create/reference/feature-execution.md
# Expected: 5

# SKILL.md has 10 phases
grep -c '### Phase' .claude/plugins/sdlc/skills/define/SKILL.md
# Expected: 10

# No remaining references to /sdlc:create or /sdlc:update as standalone in SKILL.md Integration table
grep 'sdlc:create\|sdlc:update' .claude/plugins/sdlc/skills/define/SKILL.md
# Expected: no output (or only in Red Flags "don't skip" context)
```

- [ ] **Step 2: Verify unchanged files were not touched**

```bash
git diff --name-only
# Expected: only these 4 files modified (they should all be committed by now, so this should be empty)
# Verify with:
git log --oneline -4
# Expected: 4 commits from tasks 1-4
```
