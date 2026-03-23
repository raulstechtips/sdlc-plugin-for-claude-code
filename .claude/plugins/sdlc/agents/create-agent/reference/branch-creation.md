# Branch Creation Reference

Shared reference for creating and linking a Git branch to a GitHub Issue. Used by execution references (`epic-execution.md`, `feature-execution.md`, `story-execution.md`) and the `/setup-dev` skill.

## Inputs

| Variable | Description |
|----------|-------------|
| `ISSUE_NUM` | The issue number to link the branch to |
| `ISSUE_TITLE` | The issue title (used to generate the branch name slug) |
| `LEVEL` | Artifact level: `pi`, `epic`, `feature`, or `story` |
| `PARENT_ISSUE` | Parent issue number, or `none` (for epics, or when no parent exists) |

## Step 1: Resolve Parent Branch

If `PARENT_ISSUE` is `none`, set `BASE_BRANCH=main` and skip to Step 2.

Otherwise, query the parent issue's linked branches:

```bash
gh issue develop <PARENT_ISSUE> --list
```

Three cases:

- **No linked branches** — `BASE_BRANCH=main`
- **Exactly one branch** — `BASE_BRANCH=<that branch name>`
- **Multiple branches** — Present the list to the user and ask which branch to use. Set `BASE_BRANCH` to their selection.

## Step 2: Create and Link Branch

Slugify the issue title:
- Lowercase the entire title
- Replace any non-alphanumeric character with a hyphen
- Collapse consecutive hyphens into one
- Strip leading and trailing hyphens

```bash
BRANCH_NAME="<LEVEL>/<ISSUE_NUM>-<SLUGIFIED_TITLE>"

gh issue develop <ISSUE_NUM> \
  --name "$BRANCH_NAME" \
  --base <BASE_BRANCH>
```

If `gh issue develop` fails because a branch with that name already exists but is not linked to this issue, surface the error to the user and suggest either renaming or manually linking the existing branch.

## Step 3: Verify

```bash
gh issue develop <ISSUE_NUM> --list
```

Confirm the branch appears in the output. The branch will also be visible in the GitHub Issue "Development" sidebar.
