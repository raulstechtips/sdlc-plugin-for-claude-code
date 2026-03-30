# Chore Brainstorming Guide

Internal checklist for the define skill. Weave these into natural conversation — do NOT ask them as a list.

## Before you can draft, make sure you understand:

- What's the specific work to perform?
- Why does this need doing? (tech debt, cleanup, migration, infrastructure)
- What are the acceptance criteria? (specific, testable)
- What files will be created or modified?
- Are there existing patterns to follow?
- Any technical considerations or edge cases?
- What blocks this or what does this block?
- Does this chore belong to an existing PI, epic, or feature — or is it standalone?

## Context to load

- Read `.claude/sdlc/prd/PRD.md` — check Security Constraints and Architecture sections
- **If parented:** read the parent PI/epic/feature issue(s) via `gh issue view <parent> --json title,body,labels`. Load the parent's Technical Notes section if it exists — the chore's Technical Notes should build on parent context.
- **If standalone:** skip parent loading — the chore's Technical Notes captures brainstorm context independently.

## Scope note

Chores have no size/decomposition concept (unlike features with size:small/large). A chore is always a single unit of work. Chores can optionally be parented to a PI, epic, or feature, or remain standalone. Chores are NOT tracked in parent checklists — they relate to parents via the `## Parent` section only.

## Research agent

For chores with unclear file scope, dispatch a research agent:
- Find files that will be modified (exact paths)
- Identify existing patterns to follow
- Check for related tests
- Note utilities/helpers to reuse

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/chore-template.md`
