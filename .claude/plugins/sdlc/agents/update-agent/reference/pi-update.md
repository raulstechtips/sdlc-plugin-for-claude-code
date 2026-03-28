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

## Edit Pattern

### Step 0: Ensure Branch Exists

Follow [`branch-creation.md`](../create-agent/reference/branch-creation.md) with:
- `ISSUE_NUM` = `<N>` (the issue being updated)
- `ISSUE_TITLE` = `<current issue title>`
- `LEVEL` = `pi`
- `PARENT_ISSUE` = `none`

This is idempotent — if a branch is already linked, this step is a no-op.

### Read current PI

```bash
gh issue view <N> --json body,title,labels
```

### Make targeted edits

Extract the `body` field, apply string manipulation to modify only the targeted content, then write it back. Do NOT rewrite the entire body — change only the targeted section.

### Common direct updates

**Date change:**
Update the `started` or `target` field in the metadata section of the body, then:

```bash
gh issue edit <N> --body "$UPDATED_BODY"
```

**Theme tweak:**
Update the `theme` field in the metadata section of the body, then:

```bash
gh issue edit <N> --body "$UPDATED_BODY"
```

**Status update for an epic entry:**
Find the epic line in the `## Epics` section of the body, update its status annotation, then:

```bash
gh issue edit <N> --body "$UPDATED_BODY"
```

**Single epic/feature rename:**
Find the line referencing the old name, replace it with the new name (keeping the issue number reference intact), then:

```bash
gh issue edit <N> --body "$UPDATED_BODY"
```

**Title change:**

```bash
gh issue edit <N> --title "<new title>"
```

**Label change:**

```bash
gh issue edit <N> --add-label "<label>" --remove-label "<label>"
```

Use a descriptive description of what changed: `gh issue edit <N> --body "$UPDATED_BODY"` extending the target date, or renaming an epic entry.

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
