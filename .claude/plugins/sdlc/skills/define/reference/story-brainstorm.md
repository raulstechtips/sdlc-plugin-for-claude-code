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
