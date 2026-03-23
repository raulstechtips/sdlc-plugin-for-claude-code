---
name: setup-dev
description: Align a git worktree to an issue's linked branch and start development. Use when you need to set up a worktree for working on a specific GitHub issue.
---

# Setup Dev

Align a git worktree to a GitHub Issue's linked branch. Creates the branch if it doesn't exist yet.

## Step 1: Worktree Gate

Verify this is a git worktree, not the main checkout:

```bash
GIT_DIR=$(git rev-parse --git-dir)
GIT_COMMON=$(git rev-parse --git-common-dir)
test "$GIT_DIR" != "$GIT_COMMON"
```

This works from any subdirectory within the worktree.

If the test fails (values are equal), this is the main checkout. **Abort** with:

> "This skill requires a git worktree. Start one with `claude -w` and try again."

## Step 2: Parse Argument

Extract the issue number from `$ARGUMENTS`. Expected format: `#42` or `42`.

If no issue number is provided, **abort** with:

> "Usage: `/sdlc:setup-dev #<issue-number>`"

## Step 3: Resolve Variables from Issue

Query the issue to extract all needed context:

```bash
gh issue view <ISSUE_NUM> --json title,labels,body
```

Extract the following variables:

**LEVEL** — from the `type:*` label:
- `type:epic` → `epic`
- `type:feature` → `feature`
- `type:story` → `story`
- `type:chore` → `chore`
- `type:bug` → `bug`
- If no recognized `type:` label is found, **abort** with: "Cannot determine artifact level. This skill only supports epic, feature, story, chore, and bug issues."

**ISSUE_TITLE** — from the issue title.

**PARENT_ISSUE** — resolved via a unified algorithm for all types:

**Phase 1: Parse `## Parent` section.** Extract from the issue body, checking fields in priority order:
1. `Feature: #N` → candidate = N
2. `Epic: #N` → candidate = N
3. `PI: #N` → candidate = N

First match wins.

**Phase 2: Confirm or prompt.**

- **If candidate found** → confirm with the user:

  > "Found parent issue #N. Use its branch as the base? (Or provide a different branch/issue number, or press enter to branch from `main`)"

  - User confirms → `PARENT_ISSUE=<candidate>`
  - User provides alternative → resolve using the same flow as "no candidate" below
  - User presses enter → `PARENT_ISSUE=none` (branches from `main`)

- **If no candidate** (no `## Parent` section, or no recognized field) → prompt the user:

  > "No parent found for this issue. You can provide:
  > - A branch name (e.g., `feature/execution-skills-stabilization`)
  > - An issue number (e.g., `#4`) — I'll resolve its linked branch
  > - Or press enter to branch from `main`"

  Based on the user's response:

  1. **Branch name** (no `#` prefix): Verify the branch exists with `git ls-remote --heads origin <branch-name>`. If it exists, set `BASE_BRANCH=<branch-name>`. If not, warn the user and re-prompt.

  2. **Issue number** (`#N` or `N`): Run `gh issue develop <N> --list`.
     - **No linked branches** — warn: "Issue #N has no linked branch." Re-prompt.
     - **Exactly one branch** — set `BASE_BRANCH=<that branch>`.
     - **Multiple branches** — present the list and ask the user to pick one.

  3. **Empty / enter** — `PARENT_ISSUE=none` (branches from `main`).

## Step 4: Check for Existing Linked Branch

```bash
gh issue develop <ISSUE_NUM> --list
```

- **Branch exists** → store the branch name as `BRANCH_NAME`, skip to Step 6
- **No branch** → proceed to Step 5

## Step 5: Create Branch (if needed)

Follow the procedure in [`branch-creation.md`](../../agents/create-agent/reference/branch-creation.md) with the resolved variables:
- `ISSUE_NUM` — from Step 2
- `ISSUE_TITLE` — from Step 3
- `LEVEL` — from Step 3
- `PARENT_ISSUE` — from Step 3
- `BASE_BRANCH` — *(bugs only)* from Step 3's interactive resolution. When provided, branch-creation skips its own parent resolution.

After branch creation, store the resulting branch name as `BRANCH_NAME`.

## Step 6: Align Worktree

```bash
git fetch origin
git checkout <BRANCH_NAME>
```

**If `git checkout` fails** with "already checked out at '<path>'", this means another worktree is already using this branch. Surface the error to the user — this is a legitimate conflict, not something to silently resolve.

## Step 7: Update Status Label (Stories Only)

Status labels are only used on stories. Epics and features do not carry status labels.

**If `LEVEL` is not `story`, skip this step.**

Swap the story's status to `status:in-progress`:

```bash
# Remove any existing status label
gh issue edit <ISSUE_NUM> --remove-label "status:todo"
gh issue edit <ISSUE_NUM> --remove-label "status:blocked"
gh issue edit <ISSUE_NUM> --remove-label "status:done"

# Set in-progress
gh issue edit <ISSUE_NUM> --add-label "status:in-progress"
```

## Step 8: Report

Display to the user:
- Branch name
- Issue title and number
- Status: `in-progress` (stories) or current status (epics/features/chores/bugs)

Example output:

> **Setup complete:**
> - Branch: `story/42-implement-user-auth`
> - Issue: #42 — "Implement user authentication"
> - Status: `in-progress`
