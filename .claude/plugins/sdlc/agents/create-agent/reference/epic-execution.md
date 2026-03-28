# Epic Execution Reference

## Required Fields

### Frontmatter
- `name` — epic name
- `priority` — one of: `critical`, `high`, `medium`, `low`
- `areas` — array of area labels (e.g., `[auth, api]`)
- `parent-pi` — parent PI issue number

### Body Sections
- `## Overview` — non-empty
- `## Success Criteria` — at least one checkbox item
- `## Features` — at least one checklist item with `(#TBD)` placeholder

Additional sections (Non-goals, Technical Notes, Dependencies) are expected but not blocking.

## Stub Mode

If dispatched with `stub: true`, skip Step 2 (Create and Link Branch). All other steps execute normally.

## Execution Steps

### 1. Create the Epic Issue

Write the draft body (without YAML frontmatter) to a temp file and create the issue:

```bash
# Strip frontmatter from draft, write body to temp file
# (everything after the closing --- of the frontmatter)

cat <<'BODY' > /tmp/sdlc-epic-body.md
<draft body content without frontmatter>
BODY

EPIC_URL=$(gh issue create \
  --title "<name>" \
  --body-file /tmp/sdlc-epic-body.md \
  --label "type:epic" \
  --label "priority:<priority>" \
  --label "area:<area1>" \
  --label "area:<area2>")
EPIC_NUM=$(echo "$EPIC_URL" | grep -o '[0-9]*$')
```

**Note:** Do NOT add a `status:` label to epics. Status labels are for stories only.

### 2. Create and Link Branch

Follow [`branch-creation.md`](branch-creation.md) with:
- `ISSUE_NUM` = `<EPIC_NUM>`
- `ISSUE_TITLE` = `<name>`
- `LEVEL` = `epic`
- `PARENT_ISSUE` = `<parent-pi>`

### 3. Bidirectional Dependency Linking

If the epic draft has a `## Dependencies` section with `Blocked by: #N, #M`:

For each referenced issue number:

```bash
# Read the blocker's body
BLOCKER_BODY=$(gh issue view <N> --json body --jq '.body')

# Check if it has a Dependencies section with a "Blocks:" line
# If "Blocks: none" -> replace with "Blocks: #<EPIC_NUM>"
# If "Blocks: #X" -> append ", #<EPIC_NUM>"
# If no Dependencies section -> append one

# Write back
echo "$UPDATED_BLOCKER_BODY" | gh issue edit <N> --body-file -
```

### 4. Update Parent PI Issue

Replace the `#TBD` placeholder in the parent PI issue body with the real epic number:

```bash
PI_BODY=$(gh issue view <parent-pi> --json body --jq '.body')
UPDATED_BODY=$(echo "$PI_BODY" | sed "s/### Epic: <EPIC_NAME> (#TBD)/### Epic: <EPIC_NAME> (#<EPIC_NUM>)/")
gh issue edit <parent-pi> --body "$UPDATED_BODY"
```

### 5. Clean Up Temp Files

```bash
rm -f /tmp/sdlc-epic-body.md
```

## Report Format

> **Created:**
> - Epic: #`<EPIC_NUM>` — "`<name>`"
>   - Labels: `type:epic`, `priority:<priority>`, `area:<areas>`
>   - Branch: `epic/<EPIC_NUM>-<slugified-name>` (linked to issue) — omit if stub mode
>
> **Updated:**
> - Blocker #`<N>` body: added `Blocks: #<EPIC_NUM>` to Dependencies
> - PI issue #`<parent-pi>`: replaced `#TBD` with #`<EPIC_NUM>`
