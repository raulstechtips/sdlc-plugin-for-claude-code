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
