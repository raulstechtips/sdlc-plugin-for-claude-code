# SDLC Plugin for Claude Code

Claude Code plugin (v2.0.0) providing structured SDLC management via GitHub Issues and git-versioned artifacts.

## Project Structure

.claude/plugins/sdlc/          # Plugin source
  plugin.json                  # Manifest
  templates/                   # Shared draft templates (rigid output formats)
  skills/                      # 8 skills
    init/ capture/ define/ create/ update/ status/ reconcile/ retro/
  agents/                      # 4 agents
    draft-reviewer/ create-agent/ update-agent/ impact-analysis-agent/

.claude/sdlc/                  # SDLC artifacts (runtime data)
  prd/PRD.md                   # Product requirements (git-versioned)
  pi/PI.md                     # Current PI plan (git-versioned)
  pi/completed/                # Archived PI plans
  drafts/                      # Local working drafts (NOT committed)
  retros/                      # Retrospective documents

docs/                          # Research and design documents

## SDLC Workflow

Skills are invoked as slash commands. The lifecycle:

1. `/sdlc:init` — bootstrap labels and directories (once, after PRD exists)
2. `/sdlc:define [level]` — brainstorm and produce a local draft (level is optional — skill classifies if not specified)
3. `/sdlc:create <level>` — validate draft, publish to GitHub/git
4. `/sdlc:update <level> #N` — surgical edits to existing artifacts
5. `/sdlc:status` — read-only project briefing
6. `/sdlc:reconcile` — audit and fix label drift
7. `/sdlc:retro` — process metrics and retrospective
8. `/sdlc:capture` — quick-capture idea as triage issue

Two-phase workflow: define (brainstorm) -> create/update (execute). Define may dispatch agents for side-effect updates confirmed by the user during impact analysis.

## Conventions

- Artifact hierarchy: PRD > PI > Epic > Feature > optional Story
- Features are classified as size:small (directly implementable) or size:large (decomposed into stories)
- PRD and PI are git files; Epic/Feature/Story are GitHub Issues
- All metadata via GitHub Labels (not Projects) — Timeline API dependency
- Dependencies are bidirectional: `- Blocked by: #N` matched by `- Blocks: #N`
- Drafts live in `.claude/sdlc/drafts/` and are never committed
- Commits use conventional format: `docs(prd):`, `docs(pi):`, etc.

## Prerequisites

- `gh` CLI authenticated (`gh auth status`)
- `jq` installed
- Git repository with GitHub remote
