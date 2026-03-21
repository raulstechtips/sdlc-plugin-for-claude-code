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
