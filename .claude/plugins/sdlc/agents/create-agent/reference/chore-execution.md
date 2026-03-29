# Chore Execution Reference

## Required Fields

### Frontmatter
- `name` — chore name
- `priority` — one of: `critical`, `high`, `medium`, `low`
- `areas` — array of area labels

### Optional Frontmatter
- `parent-pi` — issue number of the parent PI
- `parent-epic` — issue number of the parent epic
- `parent-feature` — issue number of the parent feature

### Body Sections
- `## Description` — non-empty
- `## Task` — non-empty
- `## Acceptance Criteria` — at least one checkbox item

Additional sections (File Scope, Technical Notes, Dependencies, Parent) are expected but not blocking.

## Execution Steps

### 1. Create the Chore Issue

Write the draft body (without YAML frontmatter) to a temp file and create the issue with the full set of labels.

Slugify the chore name for the temp file path — lowercase, replace non-alphanumeric characters with hyphens, collapse consecutive hyphens, strip leading/trailing hyphens.

```bash
# SLUG = slugified <name> (e.g., "Migrate config to YAML" -> "migrate-config-to-yaml")

cat <<'BODY' > /tmp/sdlc-chore-<SLUG>-body.md
<draft body content without frontmatter>
BODY

CHORE_URL=$(gh issue create \
  --title "<name>" \
  --body-file /tmp/sdlc-chore-<SLUG>-body.md \
  --label "type:chore" \
  --label "priority:<priority>" \
  --label "area:<area1>" \
  --label "area:<area2>")
CHORE_NUM=$(echo "$CHORE_URL" | grep -o '[0-9]*$')
```

**Label rules:**
- `type:chore` — always
- `priority:<priority>` — from frontmatter
- `area:<area>` — one `--label` per area from frontmatter
- Do NOT add `status:` labels — chores do not carry status labels

### 2. Bidirectional Dependency Linking

If the chore draft has `Blocked by: #N, #M` in its Dependencies section:

For each referenced blocker issue number:

```bash
# Read the blocker's body
BLOCKER_BODY=$(gh issue view <N> --json body --jq '.body')

# Update the "Blocks:" line in the blocker's Dependencies section:
# - If "Blocks: none" -> replace with "Blocks: #<CHORE_NUM>"
# - If "Blocks: #X" -> change to "Blocks: #X, #<CHORE_NUM>"
# - If "Blocks: #X, #Y" -> change to "Blocks: #X, #Y, #<CHORE_NUM>"
# - If no Dependencies section exists -> append:
#     ## Dependencies
#     - Blocked by: (unchanged)
#     - Blocks: #<CHORE_NUM>

# Write back
echo "$UPDATED_BLOCKER_BODY" | gh issue edit <N> --body-file -
```

### 3. Clean Up Temp Files

```bash
rm -f /tmp/sdlc-chore-<SLUG>-body.md
```

**Note:** Chores never get branches at creation time. Use `/sdlc:setup-dev` to create a chore branch when development begins. Chores are NOT tracked in parent checklists — they relate to parents via the `## Parent` section only.

## Report Format

> **Created:**
> - Chore: #`<CHORE_NUM>` — "`<name>`"
>   - Labels: `type:chore`, `priority:<priority>`, `area:<areas>`
>
> **Updated:**
> - Blocker #`<N>` body: added `Blocks: #<CHORE_NUM>` to Dependencies
