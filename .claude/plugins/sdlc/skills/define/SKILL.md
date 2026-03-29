---
name: define
description: Use when defining new SDLC artifacts (PRD, PI, epic, feature, story, bug, chore) or reshaping existing ones through collaborative brainstorming that produces a reviewable local draft.
allowed-tools: Read, Edit, Write, Bash, Grep, Glob, Agent
argument-hint: "[level] [#N]"
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
| "I'll skip execution, the user can run /sdlc:create" | Phase 8 handles execution. Define is end-to-end now. |

## Process Flow

You MUST create a task for each of these phases and complete them in order:

### Phase 1: Load Context

Parse `$ARGUMENTS` for an optional level and optional identifier.

- If level provided: load brainstorming guide from `${CLAUDE_PLUGIN_ROOT}/skills/define/reference/<level>-brainstorm.md`
- **Exception — chore level:** load `${CLAUDE_PLUGIN_ROOT}/skills/define/reference/story-brainstorm.md` instead (no `chore-brainstorm.md` exists). Note: parent-related questions in the story guide are conditional for chores — standalone chores skip them.
- If no level: proceed without a guide — the brainstorm will discover the level
- Read `.claude/sdlc/prd/PRD.md` for project context
- Check for active PI issue: `gh issue list --label "type:pi" --state open --json number,title,body --jq '.[0]'`
- Check pre-flight requirements (see Pre-Flight Checks below)
- Detect new vs reshape:
  - **Issue number in args** → fetch the issue via `gh issue view <N> --json title,body,labels`. Then assess the body content:
    - Read the body and judge whether this is a **fully-defined artifact** (structured template sections filled with substantive content) or a **stub/incomplete issue** (minimal body, missing template sections, placeholder text like `[TBD]`, or just a short description with parent links).
    - Present your assessment to the user: "This issue looks like [a stub that needs first-time definition / a well-defined artifact — would you like to reshape it?]"
    - Route based on the user's confirmation: first-time definition → treat as **new**; reshape → treat as **reshape**.
    - Use your own judgment on content completeness — no fixed heuristic rules. The user confirmation is the safety net.
  - **Existing draft** in `.claude/sdlc/drafts/` dir for this artifact (check after the issue assessment above, if applicable) = ask user whether to resume the draft or start fresh
  - **No issue number and no existing draft** = new artifact
- If args contain an issue number (#N) with no level keyword: infer level from the fetched issue's labels (reuse the `gh issue view` result from above):
  - Has `type:bug` label → set level to `bug`, load `bug-brainstorm.md` (via the generic rule)
  - Has `type:chore` label → set level to `chore`, load `story-brainstorm.md` (via the chore exception)
  - Has `triage` label → ask the user what level this should become (may be feature, story, bug, or chore)
  - Has any other `type:*` label (e.g., `type:feature`, `type:epic`, `type:story`, `type:pi`) → infer level from the label name, load the corresponding brainstorm guide

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

Write to `.claude/sdlc/drafts/<level>-<name>.md` (new, no issue number) or `.claude/sdlc/drafts/<level>-<issue-number>.md` (reshape, or new from a stub with a known issue number). Ensure the drafts directory exists.

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

### Phase 8: Execution

Dispatch agents to create or update the primary artifact immediately. The user approved the draft in Phase 7 — execute now, before impact analysis, so downstream phases have real issue numbers.

**Decision tree:**

~~~
Is this a reshape? (draft has ## Changes section)
├── YES: Is this a reclassification? (draft type ≠ existing issue type label)
│   ├── YES (reclassification reshape):
│   │   1. Dispatch update-agent: swap type label, remove stale labels (e.g., size:* if becoming epic), replace body with draft body
│   │   2. If new level has children (epic/feature-large): dispatch create-agents in parallel for children
│   │   3. If children created: dispatch update-agent to backfill parent body #TBD → real numbers
│   └── NO (normal reshape):
│       1. Dispatch update-agent with Changes table parameters
├── NO (new artifact): What level?
│   ├── PRD:
│   │   1. Dispatch create-agent (handles git add + commit)
│   ├── PI:
│   │   1. Dispatch create-agent for PI → receive PI issue number
│   │   2. Dispatch create-agents in parallel for each epic in ## Epics checklist (each gets parent-pi = new number, stub: true)
│   │   3. Collect all epic issue numbers
│   │   4. Dispatch update-agent to backfill PI body: replace each #TBD with real epic number
│   ├── Epic:
│   │   1. Dispatch create-agent for epic → receive issue number
│   │   2. Dispatch create-agents in parallel for each feature in ## Features checklist (each gets parent-epic = new number, stub: true)
│   │   3. Collect all feature issue numbers
│   │   4. Dispatch update-agent to backfill epic body: replace each #TBD with real feature number
│   ├── Feature (size:large):
│   │   1. Dispatch create-agent for feature → receive issue number
│   │   2. For each story in ## Stories checklist, evaluate whether the brainstorm produced enough context (acceptance criteria, file scope, technical notes) to write a full story body. If yes, pass a complete body; if not, pass stub: true with name + one-line description + parent links.
│   │   3. Dispatch create-agents in parallel for each story (each gets parent-feature = new number, parent-epic from draft frontmatter, and either a full body or stub: true per the evaluation above)
│   │   4. Collect all story issue numbers
│   │   5. Dispatch update-agent to backfill feature body: replace each #TBD with real story number
│   ├── Feature (size:small):
│   │   1. Dispatch create-agent for feature (no children)
│   └── Story:
│       1. Dispatch create-agent for story (no children)
~~~

**Draft cleanup:** After all Phase 8 agents complete successfully (primary created, all children created, backfill complete), delete the working draft:

```bash
rm .claude/sdlc/drafts/<draft-filename>
```

Do NOT delete the draft if any Phase 8 agent fails — the draft is the recovery artifact.

**Sequencing rule:** The primary artifact MUST be created first (sequential) because children need its issue number for their `## Parent` section. Children can then be created in parallel. The backfill step MUST wait for all children to complete.

**What define passes to agents:**

To create-agent:
- Draft file path (or inline body for stubs)
- Artifact level
- `stub: true` flag when the child is a stub — always for PI→epics and epic→features; for feature→stories only when brainstorm context is insufficient
- For child stubs: the child name, one-line description from the parent's checklist, parent issue numbers, inherited priority and area labels

To update-agent:
- Target issue number
- Level
- Change description (from Changes table for reshapes, or "backfill #TBD with #N for child-name" for backfills)
- For reclassification: old type, new type, labels to add, labels to remove, new body content

### Phase 9: Impact Analysis

The confirm-then-dispatch loop:

1. Dispatch `impact-analysis-agent` with: summary of brainstorm decisions, reclassifications, primary issue number (or PRD file path for PRD artifacts), child issue numbers (only when Phase 8 created children), relevant issue numbers
2. Agent returns structured list of impacts
3. Present impacts to the user ONE AT A TIME
4. Each impact is a mini-conversation — creative, back-and-forth, may involve follow-up questions
5. As each impact is confirmed, dispatch the appropriate operational agent (`create-agent` or `update-agent`)
6. Independent updates can run in parallel (multiple Agent dispatches in one message); dependent updates run sequentially (create issue first, then reference its number)

### Phase 10: Next Steps

Informational guidance — no dispatching, no "want me to create?". Content is level-dependent:

- **Epic** → "Features #X, #Y, #Z created as stubs. Run `/sdlc:define feature #X` to flesh one out."
- **Feature (large)** → "Stories #A, #B created as stubs. Run `/sdlc:define story #A` to flesh one out."
- **Feature (small)** → "Feature is directly implementable. Ready to develop."
- **Story** → "Next unfinished story under this feature is #B, or all stories defined — ready to develop."
- **PRD** → "Committed. Run `/sdlc:define epic` to start decomposing."
- **PI** → "Created as GitHub Issue. Run `/sdlc:define epic` to start decomposing."

## Pre-Flight Checks

| Level | Prerequisites |
|-------|--------------|
| PRD | Check if `.claude/sdlc/prd/PRD.md` exists (greenfield vs brownfield vs reshape) |
| PI | PRD exists. Check for previous closed PIs via `gh issue list --label "type:pi" --state closed`. Check for active PI issue via `gh issue list --label "type:pi" --state open`. |
| Epic | PRD exists. PI exists. |
| Feature | PRD exists. PI exists. Parent epic resolvable via `gh issue view`. |
| Story | PRD exists. Parent feature and parent epic resolvable via `gh issue view`. |
| Bug | PRD exists. No parent requirements — bugs are peers to the hierarchy. |
| Chore | PRD exists. If parented, parent epic/feature resolvable via `gh issue view`. |

If prerequisites are missing, tell the user what needs to exist first and suggest the appropriate `/sdlc:define` invocation.

## Dependency Format

All dependency references in drafts use this exact format:

- Blocked by: #N, #M
- Blocks: #N, #M

Rules: dash-prefixed, `#` prefix on issue numbers, comma-space separated, `none` when empty. Bidirectional — reconcile enforces consistency.

## Integration

| Scenario | Flow |
|----------|------|
| New artifact | define brainstorms → draft → Phase 8 dispatches create-agent (and create-agents for children if applicable) |
| Reshape existing | define brainstorms → draft with Changes section → Phase 8 dispatches update-agent |
| Side-effect updates | Phase 9 (Impact Analysis) dispatches create-agent/update-agent for confirmed cascading changes |
