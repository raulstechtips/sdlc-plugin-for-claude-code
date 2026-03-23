# Bug Brainstorming Guide

Internal checklist for the define skill. Weave these into natural conversation — do NOT ask them as a list.

## Before you can draft, make sure you understand:

- What exactly is broken? (symptoms, error messages, behavior)
- What's the expected behavior vs actual behavior?
- Can it be reproduced? What are the steps?
- Is this a regression or long-standing? (check git history)
- What's the severity? (critical/high/medium/low)
- Which areas of the system are affected?
- Are there related bugs or issues?
- What's the fix scope? (one file, multiple areas, architectural)
- Does this block or get blocked by any existing work?

## Context to load

- Search codebase for files related to the bug's symptoms
- Check recent git history for changes in affected areas
- Check open issues for related work: `gh issue list --label "type:bug"`
- Read `.claude/sdlc/prd/PRD.md` — check Architecture and Security Constraints if relevant

## Scope note

Bugs have no size/decomposition concept (unlike features with size:small/large). A bug is always a single unit of work. There is no parent to load — bugs are peers to the hierarchy.

## Research agent

For bugs with unclear scope, dispatch a research agent:
- Trace the code path where the bug manifests
- Identify all files in the affected area
- Check for related test coverage
- Look for similar patterns elsewhere that might have the same bug

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/bug-template.md`
