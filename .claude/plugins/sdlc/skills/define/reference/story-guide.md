# Story Reference Guide

## Scope Assessment Criteria

| Signal | LIGHT | STANDARD | DEEP |
|--------|-------|----------|------|
| File count | 1 file | 2-3 files | 4+ files across multiple areas |
| Pattern novelty | Clear pattern to follow in codebase | Extends existing patterns | Requires new patterns not in codebase |
| Blockers | 0 | 1-2 | 3+ |
| Parent detail level | Parent feature has detailed context | Parent feature has moderate context | Parent feature is sparse or this is a complex bug |
| Area span | 1 area | 1-2 areas | 3+ areas |
| Questions needed | 1-2 | 3-4 | 5+ |

**Quick rules:**
- DEEP if ANY: touches 4+ files across multiple areas, requires new patterns not in codebase, has 3+ blockers, is a bug with unclear root cause
- STANDARD if ANY: 2-3 files, extends existing patterns, has 1-2 blockers
- LIGHT if ALL: 1 file, clear pattern to follow, 0 blockers, parent feature already has detailed context

## Context to Read

1. **Parent feature:** `gh issue view <feature-number> --json title,body,labels` — understand the feature's scope and where this story fits.
2. **Parent epic:** `gh issue view <epic-number> --json title,body,labels` — broader epic context.
3. **PRD:** `.claude/sdlc/prd/PRD.md` — focus on:
   - **Security Constraints** (auth, data sensitivity, compliance — every story must satisfy these)
   - **Data Models** (entities and relationships this story touches)
   - **API Contracts** (endpoints this story implements or modifies)
   - **Tech Stack** (frameworks and patterns to use)

For STANDARD and DEEP stories, a research Agent is dispatched during discovery — see the "Discovery by Depth" section below for the dispatch prompt and timing.

## Question Templates

### Questions (adapt count to depth)

1. **Description:** "What needs to be built and why? (2-3 sentences — the what and the motivation)"
2. **Acceptance criteria:** "What are the testable checkboxes? (3-7 criteria that a reviewer can verify — each should be independently testable)"
3. **File scope:** "Which files will be created or modified? (exact paths — this constrains the implementation)"
4. **Technical notes:** "Any implementation patterns to follow, edge cases to handle, or existing code to reference?"
5. **Dependencies:** "What must be done before this story can start? What does completing this unblock?"

### LIGHT: Summarize from parent feature context, confirm with 1 question.
### STANDARD: Ask questions 1-3, infer 4-5 from context and codebase scan.
### DEEP: Ask all 5, plus follow-ups on edge cases, error handling, and testing strategy.

### Acceptance Criteria Guidance

Good acceptance criteria are:
- **Testable:** A reviewer can verify yes/no, not subjective
- **Specific:** Reference exact behavior, not vague quality ("returns 401" not "handles auth properly")
- **Complete:** Cover the happy path, at least one error path, and edge cases
- **Independent:** Each criterion can be verified on its own

Bad examples (avoid):
- "Code is clean" (subjective)
- "Works correctly" (vague)
- "Handles errors" (which errors? what's the behavior?)

## Draft Body Template

```markdown
---
type: story
name: <story name>
priority: <critical|high|medium|low>
areas: [<area labels>]
status: draft
parent-epic: <epic issue number>
parent-feature: <feature issue number or "none" if flat epic>
---

## Description
[What needs to be built and why — 2-3 sentences covering the functionality and its motivation within the parent feature/epic]

## Acceptance Criteria
- [ ] [testable criterion 1]
- [ ] [testable criterion 2]
- [ ] [testable criterion 3]

## File Scope
**Create:**
- `exact/path/to/new-file.py`

**Modify:**
- `exact/path/to/existing-file.py`

## Technical Notes
[Implementation patterns to follow, edge cases to handle, existing code to reference, testing approach]

## Dependencies
- Blocked by: [#N or "none"]
- Blocks: [#N or "none"]

## Parent
- Epic: #<epic number>
- Feature: #<feature number>
```

## Pre-Flight Checklist

- PRD exists at `.claude/sdlc/prd/PRD.md`
- Parent feature issue is resolvable via `gh issue view <feature-number> --json title,body,labels`
- Parent epic issue is resolvable via `gh issue view <epic-number> --json title,body,labels`

## Discovery by Depth

### LIGHT (1 file, clear pattern, 0 blockers, detailed parent context)
Summarize from parent feature context. Confirm with 1 question. Use question 1 (description) from templates above, infer acceptance criteria and file scope from parent context and codebase patterns.

### STANDARD (2-3 files, extends patterns, 1-2 blockers)
Dispatch a research Agent before asking questions:

Prompt: "Search for existing patterns related to [story topic]: 1. Find files that will be modified (exact paths) 2. Identify existing patterns to follow (similar implementations) 3. Check for related tests (test file locations, test patterns used) 4. Note any utilities or helpers that should be reused. Do not modify any files."

Then ask questions 1-3 from templates above, one at a time. Infer technical notes and dependencies from Agent findings and parent context.

### DEEP (4+ files, new patterns, 3+ blockers, sparse parent context)
Dispatch research Agent first (same prompt as STANDARD). Then ask all 5 questions from templates above, plus follow-ups on edge cases, error handling, and testing strategy.

## Approaches by Depth

### LIGHT
Present one implementation approach. Confirm with user.

### STANDARD
Present two options (e.g., different implementation strategies or patterns to follow). Recommend one.

### DEEP
Present three approaches with trade-off table (e.g., different architectural patterns, different testing strategies).

## Review Context

The draft reviewer should check this draft against:
- `.claude/sdlc/prd/PRD.md` — security constraints (auth, data sensitivity), data models, API contracts, tech stack
- Parent feature via `gh issue view #<feature-number>` — story fits within feature scope
- Parent epic via `gh issue view #<epic-number>` — story aligns with epic's success criteria
- Acceptance criteria are testable, specific, and complete (not vague — per the AC guidance in this guide)
