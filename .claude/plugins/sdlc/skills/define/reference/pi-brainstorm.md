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
- Check for previous PI plans: `gh issue list --label "type:pi" --state closed --json number,title`
- Check for active PI: `gh issue list --label "type:pi" --state open --json number,title,body --jq '.[0]'`

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/pi-template.md`
