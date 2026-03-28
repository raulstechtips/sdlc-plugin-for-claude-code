---
name: SDLC Plugin for Claude Code
version: 1.1
created: 2026-03-20
---

## Overview

SDLC Plugin for Claude Code is a Claude Code plugin that provides structured software development lifecycle management through collaborative brainstorming, GitHub Issues, and git-versioned artifacts. It enables developers to plan, decompose, execute, track, and ship work using a hierarchical work-item model (PRD > PI > Epic > Feature > Story) with process discipline enforced through mandatory phases and hard gates.

The plugin splits source of truth strategically: the PRD owns product requirements (git-versioned), and GitHub Issues own all planning and work artifacts — PI Plans, Epics, Features, and Stories — with acceptance criteria, dependencies, and status tracking. All process metrics are derived from GitHub's Timeline API via label change events, enabling data-driven process insights.

## Tech Stack

- **Runtime:** Claude Code plugin system (markdown-based skills with YAML frontmatter)
- **External CLI tools:** `gh` (GitHub CLI), `git`, `jq`, `bash`
- **Artifact storage:** Git-versioned markdown files (PRD) + GitHub Issues (PI, Epic, Feature, Story)
- **Metadata tracking:** GitHub Labels (status, priority, type, area, triage)
- **Process metrics:** GitHub Timeline API (label change events, PR events)
- **Plugin manifest:** `plugin.json` (name, description, version)

## Architecture

The plugin is a collection of 7 skills and 4 agents organized under `.claude/plugins/sdlc/`, each skill with a `SKILL.md` defining behavior and each agent with an `AGENT.md` and optional `reference/` guides containing level-specific execution instructions.

**Define-driven artifact workflow:**
1. **Brainstorm:** `sdlc:define` collaborates with the user, producing a local draft file in `.claude/sdlc/drafts/`
2. **Execute:** Define dispatches the create-agent or update-agent to publish artifacts to GitHub Issues or git — no separate skill invocation needed
3. **Impact analysis:** Define identifies cascading updates and dispatches agents for confirmed side-effects

**Artifact hierarchy (top-down decomposition):**
```
PRD (git: .claude/sdlc/prd/PRD.md)
  └── PI Plan (GitHub Issue, type:pi label)
       └── Epic (GitHub Issue, type:epic)
            └── Feature (GitHub Issue, type:feature)
                 └── Story (GitHub Issue, type:story)
```

**Skill interaction patterns:**
- `sdlc:init` bootstraps infrastructure (labels, directories) by reading the PRD
- `sdlc:define` brainstorms, produces drafts, and dispatches create-agent or update-agent for execution
- `sdlc:status` is read-only — gathers state from GitHub Issues and PI Plan
- `sdlc:reconcile` audits and fixes label drift across all open issues
- `sdlc:capture` creates typed work items (bug, chore, triage) with dedicated templates
- `sdlc:setup-dev` aligns a worktree to an issue's linked branch for development
- `sdlc:finish-dev` creates a PR to merge a roll-up branch into its parent branch

**Key architectural decisions:**
- Labels-only approach (no GitHub Projects) — Project field changes don't appear in the Timeline API, which would break label-based status tracking
- Bidirectional dependency linking — every "Blocked by" in issue A is matched by a "Blocks" in issue B
- Creative brainstorming with lightweight checklists — define skill uses free-form conversation guided by internal checklists, replacing the rigid depth-based question system
- Define owns the full lifecycle — brainstorm, draft, execute, impact analysis. Define dispatches create-agent and update-agent directly, eliminating the separate create/update skill handoff

## Data Models

### Artifact Types

| Artifact | Storage | Key Fields |
|----------|---------|-----------|
| PRD | `.claude/sdlc/prd/PRD.md` (git) | name, version, created, Overview, Tech Stack, Architecture, Data Models, API Contracts, Security Constraints, Roadmap, Acceptance Criteria, Out of Scope, Label Taxonomy, Decision Log |
| PI | GitHub Issue (`type:pi`) | name, theme, Timeline (started, target), Goals, Epics (with scope seeds and #TBD placeholders), Dependencies, Worktree Strategy |
| Epic | GitHub Issue (`type:epic`) | title, Overview, Success Criteria, Features checklist, Non-goals, Technical Notes, Dependencies |
| Feature | GitHub Issue (`type:feature`) | title, Description, size (small/large), Acceptance Criteria, Stories checklist (if size:large), Non-goals, Technical Notes, Dependencies, Parent Epic link |
| Story | GitHub Issue (`type:story`) | title, Description, Acceptance Criteria, File Scope, Technical Notes, Dependencies, Parent Epic + Feature links |

### Label Taxonomy

| Category | Labels | Purpose |
|----------|--------|---------|
| Type | `type:pi`, `type:epic`, `type:feature`, `type:story`, `type:bug`, `type:chore` | Classify work item kind |
| Status | `status:todo`, `status:in-progress`, `status:done`, `status:blocked` | Track workflow state |
| Priority | `priority:critical`, `priority:high`, `priority:medium`, `priority:low` | Rank urgency |
| Area | `area:skills`, `area:agents`, `area:templates`, `area:reference`, `area:manifest`, `area:artifacts`, `area:docs` | Group by architectural area |
| Triage | `triage` | Unprocessed captures awaiting definition |
| Size | `size:small`, `size:large` | Classify feature complexity — small features are directly implementable, large features decompose into stories |

### Dependency Format (canonical)

```markdown
- Blocked by: #N, #M
- Blocks: #N, #M
```

Rules: dash-prefixed, `#` prefix on issue numbers, comma-space separated, `none` when empty. Bidirectional — reconcile enforces consistency.

### Parent Completion Rules

- PI closes when all Epics are `status:done`
- Epic closes when all Features are `status:done`
- Feature closes when all Stories are `status:done`
- Open bugs/chores tied to a parent count as open children for completion purposes
- Blocker is satisfied when issue is CLOSED or has `status:done`
- Story with unmet blockers gets `status:blocked` (enforced by reconcile)

## API Contracts

The plugin interacts with external systems exclusively through CLI tools:

### GitHub CLI (`gh`)

| Operation | Command Pattern | Used By |
|-----------|----------------|---------|
| Create issue | `gh issue create --title "..." --body "..." --label "..."` | create |
| Create PI issue | `gh issue create --label type:pi` | create (PI) |
| Close PI issue | `gh issue close <number>` | create (PI archival) |
| Edit PI issue | `gh issue edit <number> --body "..."` | update (PI) |
| View issue | `gh issue view <number> --json title,body,labels,state` | define, update, status, reconcile |
| Edit issue | `gh issue edit <number> --add-label/--remove-label/--body "..."` | update, reconcile |
| List issues | `gh issue list --label "..." --state open --json number,title,labels` | status, reconcile |
| Close issue | `gh issue close <number>` | reconcile |
| Create label | `gh label create "name" --description "..." --color "..."` | init |

### Git

| Operation | Command Pattern | Used By |
|-----------|----------------|---------|
| Commit artifact | `git add <file> && git commit -m "..."` | create (PRD) |

## Security Constraints

- **Authentication:** Delegates to `gh` CLI's existing auth (`gh auth login`). No additional token management.
- **Data sensitivity:** Issue content is stored on GitHub — inherits repository visibility settings (public/private).
- **Access control:** GitHub repository permissions govern who can create/edit issues and labels. No plugin-level access control layer.
- **No secrets handling:** The plugin does not store, transmit, or process credentials, API keys, or sensitive data beyond what `gh` manages.

## Skill Inventory

| Skill | Purpose | Phase | Modifies State? |
|-------|---------|-------|-----------------|
| `sdlc:init` | Bootstrap labels, directories, validate setup | Setup | Yes (labels, directories) |
| `sdlc:capture` | Type-aware capture of bugs, chores, and triage items with dedicated templates | Capture | Yes (creates issue) |
| `sdlc:define` | Collaborative brainstorming, draft production, and artifact execution via agent dispatch | Planning + Execution | Yes (draft file, issues, git files) |
| `sdlc:status` | Read-only project briefing with next actions | Monitoring | No |
| `sdlc:reconcile` | Audit and fix label drift | Maintenance | Yes (labels, open/closed state) |
| `sdlc:setup-dev` | Align worktree to issue branch and start development | Execution | Yes (git checkout, status label) |
| `sdlc:finish-dev` | Create PR to merge roll-up branch into parent branch | Shipping | Yes (creates PR) |

**Skill interaction flow:**
```
sdlc:init (once) → sdlc:define (brainstorm + execute) → sdlc:setup-dev → work → sdlc:finish-dev
                    sdlc:status / sdlc:reconcile (monitoring)
                    sdlc:capture (bug/chore/triage) → sdlc:define (later, for promotion)
```

## Roadmap

1. **Bug fixes and stabilization** — Ensure all 7 core skills work correctly end-to-end through a full lifecycle (define PRD → plan PI → decompose → track → reconcile → ship)
2. **Missing update reference guides** — Add pi-update.md, epic-update.md, feature-update.md, story-update.md
3. **`sdlc:audit`** — Technical verification skill that compares shipped code against PRD/PI/epic/feature/story specs using recursive subagent chains
4. **PI Changelog** — Append-only invocation tracking for `sdlc:update` and `sdlc:define` changes, enabling richer process insights
5. ~~Agents~~ — ✅ Four operational agents (impact-analysis, create-agent, update-agent, draft-reviewer) dispatched by define
6. **Retrospective analysis** — Future skill for data-driven process retrospectives using Timeline API metrics (deferred from v1)

## Acceptance Criteria

- [ ] `sdlc:init` creates all labels from PRD taxonomy and all required directories without errors
- [ ] `sdlc:define` supports creative brainstorming, draft production, artifact execution via agent dispatch, and impact analysis for each artifact level
- [ ] `sdlc:status` produces accurate briefings showing in-progress, blocked, ready, and parallelizable work
- [ ] `sdlc:reconcile` detects and fixes stale labels, unclosed parents, and dependency mismatches
- [ ] `sdlc:capture` creates typed issues (bug, chore, triage) with dedicated templates from quick input
- [ ] `sdlc:setup-dev` aligns a worktree to an issue's linked branch and starts development
- [ ] `sdlc:finish-dev` creates PRs with child-issue summaries and context-aware test plans
- [ ] Full lifecycle can be run end-to-end: PRD → PI → Epics → Features → Stories → Setup-dev → Work → Finish-dev → Reconcile
- [ ] `size:small` features can be created without a Stories section
- [ ] `size:large` features require at least one story
- [ ] Impact analysis correctly identifies cascading updates to PI, parent issues, and PRD
- [ ] Operational agents (create-agent, update-agent) successfully execute dispatched work

## Out of Scope

- **GitHub Projects integration** — Decided against; Project field changes don't appear in the Timeline API, breaking label-based tracking
- **`sdlc:audit`** — Future skill for technical verification against specs
- **PI Changelog** — Future append-only invocation tracking mechanism
- **GUI/web interface** — Plugin operates entirely through Claude Code CLI
- **Non-GitHub issue trackers** — No support for Linear, Jira, Azure DevOps, or other platforms
- **Multi-repo support** — Plugin operates within a single repository context

## Decision Log

| Date | Decision | Reason | Affects |
|------|----------|--------|---------|
| 2026-03-20 | Labels-only metadata (no GitHub Projects) | Project field changes don't appear in Timeline API, which would break label-based tracking that depends on label change events | Architecture, Reconcile, Status |
| 2026-03-20 | ~~Git-versioned PRD and PI (not GitHub Issues)~~ | ~~PRD and PI are stable documents that benefit from version control, diffing, and archival — unlike work items which need GitHub's collaboration features~~ — **PI portion superseded** by PI-to-issue migration (see 2026-03-23) | Architecture, Create, Update |
| 2026-03-20 | Bidirectional dependency linking | Enables reliable blocker detection and root-cause tracing in status/reconcile without needing to scan all issues | Data Models, Create, Update, Reconcile |
| 2026-03-20 | Two-phase artifact workflow (define → create/update) | Separates creative brainstorming from execution, preventing half-formed artifacts from reaching GitHub | Architecture, Define, Create, Update |
| 2026-03-20 | ~~Depth-based discovery (LIGHT/STANDARD/DEEP)~~ | ~~Prevents over-engineering simple artifacts~~ — **Superseded** by creative brainstorming (see below) | Define |
| 2026-03-21 | Creative brainstorming replaces depth system | Depth-based discovery made brainstorming feel rigid and formulaic; creative freedom with lightweight checklists produces better artifacts | Define, Reference Guides |
| 2026-03-21 | Flexible artifact hierarchy (optional stories) | Features don't always need stories — small features are directly implementable, simplifying the workflow | Define, Create, Feature Template |
| 2026-03-21 | ~~Amended two-phase principle with impact analysis~~ | ~~Define dispatches operational agents for side-effect artifacts (PI updates, parent updates) during impact analysis; primary artifact still follows define → create~~ — **Superseded** by define-absorbs-execution (see below) | Define, Create, Update, Agents |
| 2026-03-21 | Size labels for features (size:small, size:large) | Visible classification of feature complexity enables better planning, validation, and reconciliation | Labels, Init, Reconcile, Feature Execution |
| 2026-03-21 | Concrete area labels replacing placeholder | Area labels map to architectural zones of the plugin: skills, agents, templates, reference guides, manifests, runtime artifacts, and docs — chosen because work naturally clusters around these boundaries and each has a distinct change cadence | Labels, Init, Status, Reconcile |
| 2026-03-22 | Define absorbs execution — create/update skills removed | Simpler workflow with fewer handoffs; define dispatches agents directly so artifacts get real issue numbers before impact analysis, eliminating the separate create/update skill invocation | Architecture, Skill Inventory, plugin.json, PI Goals |
| 2026-03-23 | PI migrated from git file to GitHub Issue | Unifies artifact model, closes branch hierarchy gap (main → PI → epic → feature → story), enables epic stub creation during PI define | Architecture, Create, Update, Init, Status |
| 2026-03-23 | Remove type:spike from label taxonomy | Spike concept is unused — exploratory work is tracked via triage capture and promoted through define; a dedicated spike type adds complexity without value | Labels, Init, SDLC Guide |
| 2026-03-23 | Remove retro skill; add finish-dev | Retro requires substantial Timeline API infrastructure with limited near-term value; finish-dev fills the critical gap of closing roll-up branches with proper PRs | Skill Inventory, Architecture, Roadmap |
| 2026-03-28 | Branches created at define/develop time, not at stub creation; stories only get branches via setup-dev | Stub issues are placeholders — creating branches at stub time produces premature, likely-renamed branches with no associated work; setup-dev is the explicit signal that development is starting | Define (Phase 8), Execution References, Update References, setup-dev |
