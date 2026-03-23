# PRD Brainstorming Guide

Internal checklist for the define skill. Weave these into natural conversation — do NOT ask them as a list.

## Before you can draft, make sure you understand:

- What is this project and why does it exist?
- What's the tech stack?
- What's the high-level architecture?
- What are the architectural areas of the project? (Derive `area:*` labels from the Architecture section — e.g., if the architecture describes skills, agents, and templates as distinct zones, those become `area:skills`, `area:agents`, `area:templates`)
- What are the key data models?
- What are the API contracts or interfaces?
- What are the security constraints?
- What's the roadmap (high-level phases)?
- What's in scope and out of scope?
- What decisions have already been made and why?

## Special handling

**Brownfield projects:** Before asking questions, dispatch a research agent to analyze the existing codebase:
- Tech stack (languages, frameworks, package managers)
- Directory structure (top-level layout, key directories)
- Architecture patterns
- Data models
- API endpoints
- Security patterns

Use the research findings to pre-fill what you can and focus questions on what the codebase can't tell you (goals, roadmap, constraints, decisions).

## Template reference

Draft output format: `${CLAUDE_PLUGIN_ROOT}/templates/prd-template.md`
