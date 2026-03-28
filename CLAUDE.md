# SDLC Plugin for Claude Code

Claude Code plugin (v0.3.0) providing structured SDLC management via GitHub Issues and git-versioned artifacts.

## Project Structure

.claude/plugins/sdlc/          # Plugin source
  .claude-plugin/plugin.json   # Manifest (marketplace convention)
  README.md                    # Plugin documentation
  LICENSE                      # MIT license
  templates/                   # Shared draft templates (rigid output formats)
  skills/                      # 7 skills
    init/ capture/ define/ status/ reconcile/ setup-dev/ finish-dev/
  agents/                      # 4 agents
    draft-reviewer/ create-agent/ update-agent/ impact-analysis-agent/

.claude/sdlc/                  # SDLC artifacts (runtime data)
  prd/PRD.md                   # Product requirements (git-versioned)
  drafts/                      # Local working drafts (NOT committed)

docs/                          # Research and design documents

## SDLC Workflow

Skills are invoked as slash commands. The lifecycle:

1. `/sdlc:init` — bootstrap labels and directories (once, after PRD exists)
2. `/sdlc:define [level] [#N]` — brainstorm, draft, and execute artifacts (dispatches create-agent/update-agent). When given an issue number, assesses whether the issue needs first-time definition or reshaping.
3. `/sdlc:status` — read-only project briefing
4. `/sdlc:reconcile` — audit and fix label drift
5. `/sdlc:capture` — type-aware capture of bugs, chores, and triage items
6. `/sdlc:setup-dev #N` — align worktree to issue branch, start development
7. `/sdlc:finish-dev` — create PR to merge roll-up branch into parent

Define handles the full lifecycle: brainstorm → draft → execute → impact analysis. Capture creates typed work items with light ceremony.

## Conventions

- Artifact hierarchy: PRD > PI > Epic > Feature > optional Story
- Features are classified as size:small (directly implementable) or size:large (decomposed into stories)
- PRD is a git file; PI/Epic/Feature/Story are GitHub Issues
- All metadata via GitHub Labels (not Projects) — Timeline API dependency
- Dependencies are bidirectional: `- Blocked by: #N` matched by `- Blocks: #N`
- Drafts live in `.claude/sdlc/drafts/` and are never committed
- Commits use conventional format: `docs(prd):`, etc.

## Prerequisites

- `gh` CLI authenticated (`gh auth status`)
- `jq` installed
- Git repository with GitHub remote
