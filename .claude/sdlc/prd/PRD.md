---
name: SDLC Plugin for Claude Code
version: 1.0
created: 2026-03-20
---

## Overview

SDLC Plugin for Claude Code is a Claude Code plugin that provides structured software development lifecycle management through collaborative brainstorming, GitHub Issues, and git-versioned artifacts. It enables developers to plan, decompose, execute, track, and retrospect on work using a hierarchical work-item model (PRD > PI > Epic > Feature > Story) with process discipline enforced through mandatory phases and hard gates.

The plugin splits source of truth strategically: the PRD owns product requirements (git-versioned), the PI Plan owns sprint structure (git-versioned), and GitHub Issues own detailed work items with acceptance criteria, dependencies, and status tracking. All process metrics are derived from GitHub's Timeline API via label change events, enabling data-driven retrospectives.

## Tech Stack

- **Runtime:** Claude Code plugin system (markdown-based skills with YAML frontmatter)
- **External CLI tools:** `gh` (GitHub CLI), `git`, `jq`, `bash`
- **Artifact storage:** Git-versioned markdown files (PRD, PI) + GitHub Issues (Epic, Feature, Story)
- **Metadata tracking:** GitHub Labels (status, priority, type, area, triage)
- **Process metrics:** GitHub Timeline API (label change events, PR events)
- **Plugin manifest:** `plugin.json` (name, description, version)

## Architecture

The plugin is a collection of 8 skills organized under `.claude/plugins/sdlc/`, each with a `SKILL.md` defining behavior and optional `reference/` guides containing level-specific templates and criteria.

**Two-phase artifact workflow:**
1. **Define phase:** `sdlc:define <level>` brainstorms collaboratively with the user, producing a local draft file in `.claude/sdlc/drafts/`
2. **Execute phase:** `sdlc:create <level>` or `sdlc:update <level>` validates the draft and publishes to GitHub Issues or git — no creative decisions at this stage

**Artifact hierarchy (top-down decomposition):**
```
PRD (git: .claude/sdlc/prd/PRD.md)
  └── PI Plan (git: .claude/sdlc/pi/PI.md)
       └── Epic (GitHub Issue, type:epic)
            └── Feature (GitHub Issue, type:feature)
                 └── Story (GitHub Issue, type:story)
```

**Skill interaction patterns:**
- `sdlc:init` bootstraps infrastructure (labels, directories) by reading the PRD
- `sdlc:define` produces drafts consumed by `sdlc:create` (new) or `sdlc:update` (reshape)
- `sdlc:status` is read-only — gathers state from GitHub Issues and PI Plan
- `sdlc:reconcile` audits and fixes label drift across all open issues
- `sdlc:retro` analyzes completed work using Timeline API metrics

**Key architectural decisions:**
- Labels-only approach (no GitHub Projects) — Project field changes don't appear in the Timeline API, which would break retrospective metrics
- Bidirectional dependency linking — every "Blocked by" in issue A is matched by a "Blocks" in issue B
- Depth-based discovery — artifact complexity assessed via objective criteria (file count, area span, novelty) determines question depth (LIGHT/STANDARD/DEEP)

## Data Models

### Artifact Types

| Artifact | Storage | Key Fields |
|----------|---------|-----------|
| PRD | `.claude/sdlc/prd/PRD.md` (git) | name, version, created, Overview, Tech Stack, Architecture, Data Models, API Contracts, Security Constraints, Roadmap, Acceptance Criteria, Out of Scope, Label Taxonomy, Decision Log |
| PI | `.claude/sdlc/pi/PI.md` (git, archived to `completed/PI-N.md`) | name, theme, started, target, Goals, Epics (with feature lists), Dependency Graph, Worktree Strategy |
| Epic | GitHub Issue (`type:epic`) | title, Overview, Success Criteria, Features checklist, Non-goals, Dependencies |
| Feature | GitHub Issue (`type:feature`) | title, Description, Stories checklist, Non-goals, Dependencies, Parent Epic link |
| Story | GitHub Issue (`type:story`) | title, Description, Acceptance Criteria, File Scope, Technical Notes, Dependencies, Parent Epic + Feature links |

### Label Taxonomy

| Category | Labels | Purpose |
|----------|--------|---------|
| Type | `type:epic`, `type:feature`, `type:story`, `type:spike`, `type:bug`, `type:chore` | Classify work item kind |
| Status | `status:todo`, `status:in-progress`, `status:done`, `status:blocked` | Track workflow state |
| Priority | `priority:critical`, `priority:high`, `priority:medium`, `priority:low` | Rank urgency |
| Area | `area:<name>` (project-specific) | Group by architectural area |
| Triage | `triage` | Unprocessed captures awaiting definition |

### Dependency Format (canonical)

```markdown
- Blocked by: #N, #M
- Blocks: #N, #M
```

Rules: dash-prefixed, `#` prefix on issue numbers, comma-space separated, `none` when empty. Bidirectional — reconcile enforces consistency.

### Parent Completion Rules

- Epic closes when all Features are `status:done`
- Feature closes when all Stories are `status:done`
- Blocker is satisfied when issue is CLOSED or has `status:done`
- Story with unmet blockers gets `status:blocked` (enforced by reconcile)

## API Contracts

The plugin interacts with external systems exclusively through CLI tools:

### GitHub CLI (`gh`)

| Operation | Command Pattern | Used By |
|-----------|----------------|---------|
| Create issue | `gh issue create --title "..." --body "..." --label "..."` | create |
| View issue | `gh issue view <number> --json title,body,labels,state` | define, update, status, reconcile |
| Edit issue | `gh issue edit <number> --add-label/--remove-label/--body "..."` | update, reconcile |
| List issues | `gh issue list --label "..." --state open --json number,title,labels` | status, reconcile, retro |
| Close issue | `gh issue close <number>` | reconcile |
| Create label | `gh label create "name" --description "..." --color "..."` | init |
| Timeline API | `gh api repos/{owner}/{repo}/issues/{number}/timeline` | retro |

### Git

| Operation | Command Pattern | Used By |
|-----------|----------------|---------|
| Commit artifact | `git add <file> && git commit -m "..."` | create (PRD, PI) |
| Tag PI completion | `git tag pi-N-complete` | create (PI archival) |
| Read history | `git log --fixed-strings --grep="(#N)"` | retro |

## Security Constraints

- **Authentication:** Delegates to `gh` CLI's existing auth (`gh auth login`). No additional token management.
- **Data sensitivity:** Issue content is stored on GitHub — inherits repository visibility settings (public/private).
- **Access control:** GitHub repository permissions govern who can create/edit issues and labels. No plugin-level access control layer.
- **No secrets handling:** The plugin does not store, transmit, or process credentials, API keys, or sensitive data beyond what `gh` manages.

## Skill Inventory

| Skill | Purpose | Phase | Modifies State? |
|-------|---------|-------|-----------------|
| `sdlc:init` | Bootstrap labels, directories, validate setup | Setup | Yes (labels, directories) |
| `sdlc:capture` | Quick-capture idea as triage issue | Capture | Yes (creates issue) |
| `sdlc:define` | Collaborative brainstorming producing local draft | Planning | Yes (draft file only) |
| `sdlc:create` | Validate draft, publish to GitHub/git | Execution | Yes (issues, git files) |
| `sdlc:update` | Surgical edits to existing artifacts | Modification | Yes (issues, git files) |
| `sdlc:status` | Read-only project briefing with next actions | Monitoring | No |
| `sdlc:reconcile` | Audit and fix label drift | Maintenance | Yes (labels, open/closed state) |
| `sdlc:retro` | Process metrics and retrospective analysis | Analysis | Yes (retro file, git commit) |

**Skill interaction flow:**
```
sdlc:init (once) → sdlc:define → sdlc:create → sdlc:status / sdlc:reconcile → sdlc:retro
                    sdlc:define (reshape) → sdlc:update
                    sdlc:capture → sdlc:define (later)
```

## Roadmap

1. **Bug fixes and stabilization** — Ensure all 8 core skills work correctly end-to-end through a full lifecycle (define PRD → plan PI → decompose → track → reconcile → retro)
2. **Missing update reference guides** — Add pi-update.md, epic-update.md, feature-update.md, story-update.md
3. **`sdlc:audit`** — Technical verification skill that compares shipped code against PRD/PI/epic/feature/story specs using recursive subagent chains
4. **PI Changelog** — Append-only invocation tracking for `sdlc:update` and `sdlc:define` changes, enabling richer retrospective metrics
5. **Agents** — Autonomous agents for SDLC workflows (scope TBD)

## Acceptance Criteria

- [ ] `sdlc:init` creates all labels from PRD taxonomy and all required directories without errors
- [ ] `sdlc:define` completes all 5 phases (Context Loading, Scope Assessment, Discovery, Approaches, Draft) for each artifact level (PRD, PI, Epic, Feature, Story)
- [ ] `sdlc:create` successfully publishes drafts to GitHub Issues (Epic, Feature, Story) and git files (PRD, PI) with correct labels, parent links, and bidirectional dependencies
- [ ] `sdlc:update` applies direct edits for small changes and escalates to define for large scope changes
- [ ] `sdlc:status` produces accurate briefings showing in-progress, blocked, ready, and parallelizable work
- [ ] `sdlc:reconcile` detects and fixes stale labels, unclosed parents, and dependency mismatches
- [ ] `sdlc:retro` generates retrospective documents with accurate metrics from Timeline API data
- [ ] `sdlc:capture` creates triage issues from quick input
- [ ] Full lifecycle can be run end-to-end: PRD → PI → Epics → Features → Stories → Status → Reconcile → Retro

## Out of Scope

- **GitHub Projects integration** — Decided against; Project field changes don't appear in the Timeline API, breaking retrospective metrics
- **`sdlc:audit`** — Future skill for technical verification against specs
- **PI Changelog** — Future append-only invocation tracking mechanism
- **Agents** — Future autonomous agents for SDLC workflows
- **GUI/web interface** — Plugin operates entirely through Claude Code CLI
- **Non-GitHub issue trackers** — No support for Linear, Jira, Azure DevOps, or other platforms
- **Multi-repo support** — Plugin operates within a single repository context

## Decision Log

| Date | Decision | Reason | Affects |
|------|----------|--------|---------|
| 2026-03-20 | Labels-only metadata (no GitHub Projects) | Project field changes don't appear in Timeline API, which would break retro metrics that depend on label change events | Architecture, Reconcile, Retro, Status |
| 2026-03-20 | Git-versioned PRD and PI (not GitHub Issues) | PRD and PI are stable documents that benefit from version control, diffing, and archival — unlike work items which need GitHub's collaboration features | Architecture, Create, Update |
| 2026-03-20 | Bidirectional dependency linking | Enables reliable blocker detection and root-cause tracing in status/reconcile without needing to scan all issues | Data Models, Create, Update, Reconcile |
| 2026-03-20 | Two-phase artifact workflow (define → create/update) | Separates creative brainstorming from execution, preventing half-formed artifacts from reaching GitHub | Architecture, Define, Create, Update |
| 2026-03-20 | Depth-based discovery (LIGHT/STANDARD/DEEP) | Prevents over-engineering simple artifacts while ensuring complex ones get adequate exploration | Define |
