# SDLC Plugin for Claude Code

Structured SDLC management — plan, decompose, execute, and ship work items using GitHub Issues and git-versioned artifacts.

## What It Does

The SDLC plugin brings process discipline to Claude Code sessions. It provides a hierarchical work-item model (PRD > PI > Epic > Feature > Story) with collaborative brainstorming, GitHub Issues for tracking, and labels-only metadata for workflow state.

- **Define-driven workflow** — brainstorm, draft, review, execute, and impact-analyze artifacts in a single skill invocation
- **Source-of-truth split** — PRD lives in git (`.claude/sdlc/prd/PRD.md`); PI Plans, Epics, Features, and Stories live in GitHub Issues
- **Labels-only metadata** — all status, priority, type, and area tracking uses GitHub Labels (no Projects dependency), enabling process metrics via the Timeline API
- **Bidirectional dependencies** — every "Blocked by" is matched by a "Blocks" on the other side

## Installation

### From Marketplace

```
/plugin marketplace add raulstechtips/claude-plugins
/plugin install sdlc
```

### From Source

```
claude --plugin-dir .claude/plugins/sdlc
```

## Prerequisites

- **`gh` CLI** — authenticated (`gh auth status`)
- **`jq`** — installed and on PATH
- **Git repository** with a GitHub remote

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| Init | `/sdlc:init` | Bootstrap SDLC infrastructure — creates GitHub labels, artifact directories, and checks project setup. Run once after creating a PRD. |
| Define | `/sdlc:define [level]` | Brainstorm and produce artifacts through collaborative phases. Handles the full lifecycle: brainstorm, draft, review, execute, and impact analysis. |
| Capture | `/sdlc:capture [type:] description` | Quick-capture bugs, chores, and triage items with type detection and appropriate templates. |
| Status | `/sdlc:status [area]` | Read-only project briefing — walks the hierarchy top-down, surfaces what's in progress, blocked, and ready. |
| Reconcile | `/sdlc:reconcile [area]` | Audit and fix label drift — stale labels, unclosed parents, orphaned references, circular dependencies. |
| Setup Dev | `/sdlc:setup-dev #N` | Align a git worktree to an issue's linked branch and start development. |
| Finish Dev | `/sdlc:finish-dev` | Create a PR to merge an issue's branch into its parent branch. |

## Workflow

### Planning (top-down decomposition)

```
sdlc:define prd → sdlc:define pi → sdlc:define epic → sdlc:define feature → sdlc:define story
```

Each level is brainstormed collaboratively, drafted locally, reviewed, then published to GitHub Issues (or git for PRD). The define skill dispatches create-agent and update-agent internally — no separate create/update step needed.

### Development

```
sdlc:setup-dev #N → implement → sdlc:finish-dev
```

Setup-dev aligns a worktree to the issue's branch. Finish-dev creates a PR back to the parent branch.

### Maintenance

- **`sdlc:status`** — check what's in progress, blocked, or ready
- **`sdlc:reconcile`** — fix label drift and stale references
- **`sdlc:capture`** — quickly log bugs, chores, or ideas without full ceremony

## Artifact Hierarchy

```
PRD (git: .claude/sdlc/prd/PRD.md)
  └── PI Plan (GitHub Issue, type:pi)
       └── Epic (GitHub Issue, type:epic)
            └── Feature (GitHub Issue, type:feature)
                 └── Story (GitHub Issue, type:story)
```

- **PRD** — stable product requirements: architecture, data models, decisions
- **PI Plan** — current planning interval: goals, timeline, epic assignments
- **Features** — classified as `size:small` (directly implementable) or `size:large` (decomposed into stories)
- **Stories** — smallest unit of deliverable work with file scope and acceptance criteria

## Label Taxonomy

| Category | Labels | Purpose |
|----------|--------|---------|
| Type | `type:pi`, `type:epic`, `type:feature`, `type:story`, `type:bug`, `type:chore` | Classify work item kind |
| Status | `status:todo`, `status:in-progress`, `status:done`, `status:blocked` | Track workflow state |
| Priority | `priority:critical`, `priority:high`, `priority:medium`, `priority:low` | Rank urgency |
| Area | `area:*` (project-specific, defined in PRD) | Group by architectural area |
| Triage | `triage` | Unprocessed captures awaiting definition |
| Size | `size:small`, `size:large` | Classify feature complexity |

## License

MIT — see [LICENSE](LICENSE).
