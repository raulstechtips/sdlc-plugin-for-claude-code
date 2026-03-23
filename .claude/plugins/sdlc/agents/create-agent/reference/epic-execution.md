# Epic Execution Reference

## Required Fields

### Frontmatter
- `name` — epic name
- `priority` — one of: `critical`, `high`, `medium`, `low`
- `areas` — array of area labels (e.g., `[auth, api]`)

### Body Sections
- `## Overview` — non-empty
- `## Success Criteria` — at least one checkbox item
- `## Features` — at least one checklist item with `(#TBD)` placeholder

Additional sections (Non-goals, Dependencies) are expected but not blocking.

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

### 2. Bidirectional Dependency Linking

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

### 3. Update PI.md

Read `.claude/sdlc/pi/PI.md` and replace the `#TBD` placeholder next to this epic's name with the real issue number.

```bash
# The PI.md has lines like:
# ### Epic: Auth Setup (#TBD)
# Replace (#TBD) with (#<EPIC_NUM>) for the matching epic name
```

Write the updated PI.md back.

### 4. Commit PI.md Update

```bash
git add .claude/sdlc/pi/PI.md
git commit -m "docs(pi): add issue numbers for epic <name> (#<EPIC_NUM>)"
```

### 5. Clean Up Temp Files

```bash
rm -f /tmp/sdlc-epic-body.md
```

## Report Format

> **Created:**
> - Epic: #`<EPIC_NUM>` — "`<name>`"
>   - Labels: `type:epic`, `priority:<priority>`, `area:<areas>`
>
> **Updated:**
> - Blocker #`<N>` body: added `Blocks: #<EPIC_NUM>` to Dependencies
> - PI.md: replaced `#TBD` with #`<EPIC_NUM>`
> - Committed: `docs(pi): add issue numbers for epic <name> (#<EPIC_NUM>)`
