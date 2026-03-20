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
6. **Write draft** — Use guide's draft body template. Write to `.claude/sdlc/drafts/<level>-<name>.md` (new) or `.claude/sdlc/drafts/<level>-<issue-number>.md` (reshape). Frontmatter fields vary by level — follow the guide's draft body template exactly. Reshapes MUST include a `## Changes` section documenting what was modified. Ensure `.claude/sdlc/drafts/` directory exists.
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
