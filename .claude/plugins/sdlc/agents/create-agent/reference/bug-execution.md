# Bug Execution Reference

## Required Fields

### Frontmatter
- `name` — bug name
- `severity` — one of: `critical`, `high`, `medium`, `low`
- `priority` — one of: `critical`, `high`, `medium`, `low`
- `areas` — array of area labels

### Optional Frontmatter
- `parent-pi` — issue number of the parent PI
- `parent-epic` — issue number of the parent epic
- `parent-feature` — issue number of the parent feature

### Body Sections
- `## Description` — non-empty
- `## Reproduction Steps` — non-empty (or "Not yet reproduced" with context)

Additional sections (Expected vs Actual Behavior, Affected Areas, File Scope, Technical Notes, Dependencies, Parent) are expected but not blocking.

## Execution Steps

### 1. Create the Bug Issue

Write the draft body (without YAML frontmatter) to a temp file and create the issue with the full set of labels.

Slugify the bug name for the temp file path — lowercase, replace non-alphanumeric characters with hyphens, collapse consecutive hyphens, strip leading/trailing hyphens.

```bash
# SLUG = slugified <name> (e.g., "Draft reviewer rejects valid bugs" -> "draft-reviewer-rejects-valid-bugs")

cat <<'BODY' > /tmp/sdlc-bug-<SLUG>-body.md
<draft body content without frontmatter>
BODY

BUG_URL=$(gh issue create \
  --title "<name>" \
  --body-file /tmp/sdlc-bug-<SLUG>-body.md \
  --label "type:bug" \
  --label "severity:<severity>" \
  --label "priority:<priority>" \
  --label "area:<area1>" \
  --label "area:<area2>")
BUG_NUM=$(echo "$BUG_URL" | grep -o '[0-9]*$')
```

**Label rules:**
- `type:bug` — always
- `severity:<severity>` — from frontmatter
- `priority:<priority>` — from frontmatter
- `area:<area>` — one `--label` per area from frontmatter
- Do NOT add `status:` labels — bugs do not carry status labels

### 2. Bidirectional Dependency Linking

If the bug draft has `Blocked by: #N, #M` in its Dependencies section:

For each referenced blocker issue number:

```bash
# Read the blocker's body
BLOCKER_BODY=$(gh issue view <N> --json body --jq '.body')

# Update the "Blocks:" line in the blocker's Dependencies section:
# - If "Blocks: none" -> replace with "Blocks: #<BUG_NUM>"
# - If "Blocks: #X" -> change to "Blocks: #X, #<BUG_NUM>"
# - If "Blocks: #X, #Y" -> change to "Blocks: #X, #Y, #<BUG_NUM>"
# - If no Dependencies section exists -> append:
#     ## Dependencies
#     - Blocked by: (unchanged)
#     - Blocks: #<BUG_NUM>

# Write back
echo "$UPDATED_BLOCKER_BODY" | gh issue edit <N> --body-file -
```

### 3. Clean Up Temp Files

```bash
rm -f /tmp/sdlc-bug-<SLUG>-body.md
```

**Note:** Bugs never get branches at creation time. Use `/sdlc:setup-dev` to create a bug branch when development begins. Bugs are NOT tracked in parent checklists — they relate to parents via the `## Parent` section only.

## Report Format

> **Created:**
> - Bug: #`<BUG_NUM>` — "`<name>`"
>   - Labels: `type:bug`, `severity:<severity>`, `priority:<priority>`, `area:<areas>`
>
> **Updated:**
> - Blocker #`<N>` body: added `Blocks: #<BUG_NUM>` to Dependencies
