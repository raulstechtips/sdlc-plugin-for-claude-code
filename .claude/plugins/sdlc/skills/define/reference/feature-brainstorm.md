# Feature Brainstorming Guide

Internal checklist for the define skill. Weave these into natural conversation — do NOT ask them as a list.

## Before you can draft, make sure you understand:

- What does this feature do? (description, not implementation)
- What's the acceptance criteria?
- Is this directly implementable (size:small) or does it need decomposition (size:large)?
- If large: what are the stories?
- What's NOT in scope for this feature?
- Any dependencies on other features or stories?
- Which parent epic does this belong to?
- If small: what files will be created or modified?
- What technical considerations, patterns, or approaches apply?

## Context to load

- Read the parent epic: `gh issue view <parent-epic> --json title,body,labels` — review its Technical Notes section for architectural decisions and constraints to build on
- Read active PI issue (`gh issue list --label "type:pi" --state open`) — check if this feature was listed as a scope seed
- Read `.claude/sdlc/prd/PRD.md` — check relevant Architecture and Data Models sections

## Size classification

- **size:small** — Directly implementable. No stories needed. One person, one or two sessions. Clear file scope.
- **size:large** — Needs decomposition into stories. Multiple sessions, multiple areas, or multiple people.

The brainstorm should naturally discover which size fits. Don't force the classification — let it emerge from the conversation.

## Technical considerations

After size is determined, explore technical aspects with size-aware focus:

- **size:small** — Implementation details: file scope, patterns to follow, edge cases, error handling
- **size:large** — Architectural decisions: chosen approaches, trade-offs considered, patterns that child stories need to inherit

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/feature-template.md`
