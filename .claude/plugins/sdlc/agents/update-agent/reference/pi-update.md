# PI Update Reference

## Direct Update Criteria

**DIRECT** if all true:
- 1-2 fields changing (date change, theme tweak, status update, single epic/feature rename)
- No epics added or removed
- No dependency graph restructure
- No feature redistribution between epics

**ESCALATE** if any true:
- Epic added or removed from the plan
- Dependency graph restructure (reordering epic dependencies, changing critical path)
- Feature redistribution (moving features between epics)
- 3+ fields changing
- Goal rewrite

## Read-Modify-Write Pattern

**The `gh issue edit --body` flag replaces the ENTIRE body.** There is no partial edit. Always use the read-modify-write pattern.

### Step 0: Ensure Branch Exists

Follow [`branch-creation.md`](../create-agent/reference/branch-creation.md) with:
- `ISSUE_NUM` = `<N>` (the issue being updated)
- `ISSUE_TITLE` = `<current issue title>`
- `LEVEL` = `pi`
- `PARENT_ISSUE` = `none`

This is idempotent — if a branch is already linked, this step is a no-op.

### Step 1: Read current body

Slugify the current issue title for the temp file path — lowercase, replace non-alphanumeric characters with hyphens, collapse consecutive hyphens, strip leading/trailing hyphens.

```bash
gh issue view <N> --json body --jq '.body' > /tmp/sdlc-pi-<N>-<SLUG>-body.md
```

Also read the full issue metadata for context:

```bash
gh issue view <N> --json number,title,body,labels,state
```

### Step 2: Modify the specific section

Read `/tmp/sdlc-pi-<N>-<SLUG>-body.md`, identify the section to change, and modify ONLY that section.

Common sections in a PI body:
- `## Goals` — objectives for the increment
- `## Timeline` — start/target dates
- `## Epics` — child epic subsections with status annotations

### Step 3: Write back the full updated body

```bash
gh issue edit <N> --body-file /tmp/sdlc-pi-<N>-<SLUG>-body.md
```

### Title Changes

```bash
gh issue edit <N> --title "<new title>"
```

### Label Changes

```bash
gh issue edit <N> --add-label "<label>" --remove-label "<label>"
```

### Clean Up

```bash
rm -f /tmp/sdlc-pi-<N>-<SLUG>-body.md
```

## Cascade Rules

After updating the PI, check for downstream impacts:

- **If an epic was renamed**: the corresponding GitHub issue title may need updating.

```bash
gh issue view <epic-number> --json title --jq '.title'
```

Flag if the title doesn't match the new name in the PI issue body.

- **If dates changed**: flag stories with time-sensitive dependencies.
- **If structure changed** (escalated via define, then applied): flag all affected epics and features for review.

Do not update downstream artifacts without user confirmation.
