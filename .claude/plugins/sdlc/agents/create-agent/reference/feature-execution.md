# Feature Execution Reference

## Required Fields

### Frontmatter
- `name` — feature name
- `priority` — one of: `critical`, `high`, `medium`, `low`
- `areas` — array of area labels
- `parent-epic` — issue number of the parent epic
- `size` — one of: `small`, `large`

### Body Sections
- `## Description` — non-empty
- `## Stories` — **only if `size: large`**: at least one checklist item with `(#TBD)` placeholder. **Must NOT exist if `size: small`.**

Additional sections (Non-goals, Dependencies, Parent) are expected but not blocking.

## Execution Steps

### 1. Create the Feature Issue

Write the draft body (without YAML frontmatter) to a temp file and create the issue:

```bash
cat <<'BODY' > /tmp/sdlc-feature-body.md
<draft body content without frontmatter>
BODY

FEAT_URL=$(gh issue create \
  --title "<name>" \
  --body-file /tmp/sdlc-feature-body.md \
  --label "type:feature" \
  --label "priority:<priority>" \
  --label "size:<size>" \
  --label "area:<area1>" \
  --label "area:<area2>")
FEAT_NUM=$(echo "$FEAT_URL" | grep -o '[0-9]*$')
```

**Note:** Do NOT add a `status:` label to features. Status labels are for stories only.

### 2. Create and Link Branch

Follow [`branch-creation.md`](branch-creation.md) with:
- `ISSUE_NUM` = `<FEAT_NUM>`
- `ISSUE_TITLE` = `<name>`
- `LEVEL` = `feature`
- `PARENT_ISSUE` = `<parent-epic>` (from draft frontmatter field `parent-epic`)

### 3. Update Parent Epic Body

Read the parent epic's body and replace the `#TBD` placeholder next to this feature's name with the real feature issue number.

```bash
# Read the parent epic's body
EPIC_BODY=$(gh issue view <parent-epic> --json body --jq '.body')

# Find the line with this feature's name and (#TBD), replace #TBD with #<FEAT_NUM>

# Write back
echo "$UPDATED_EPIC_BODY" | gh issue edit <parent-epic> --body-file -
```

### 4. Bidirectional Dependency Linking

If the feature draft has a `## Dependencies` section with `Blocked by: #N, #M`:

For each referenced issue number:

```bash
# Read the blocker's body
BLOCKER_BODY=$(gh issue view <N> --json body --jq '.body')

# Check if it has a Dependencies section with a "Blocks:" line
# If "Blocks: none" -> replace with "Blocks: #<FEAT_NUM>"
# If "Blocks: #X" -> append ", #<FEAT_NUM>"
# If no Dependencies section -> append one

# Write back
echo "$UPDATED_BLOCKER_BODY" | gh issue edit <N> --body-file -
```

### 5. Update PI.md (if feature is listed there)

Some PI plans list features explicitly under their epics. If `.claude/sdlc/pi/PI.md` has a `#TBD` next to this feature's name, replace it:

```bash
# Read PI.md, check if this feature's name appears with (#TBD)
# If so, replace (#TBD) with (#<FEAT_NUM>)
# Write back and commit
```

If PI.md was updated:

```bash
git add .claude/sdlc/pi/PI.md
git commit -m "docs(pi): add issue number for feature <name> (#<FEAT_NUM>)"
```

### 6. Clean Up Temp Files

```bash
rm -f /tmp/sdlc-feature-body.md
```

## Report Format

> **Created:**
> - Feature: #`<FEAT_NUM>` — "`<name>`"
>   - Labels: `type:feature`, `priority:<priority>`, `size:<size>`, `area:<areas>`
>   - Branch: `feature/<FEAT_NUM>-<slugified-name>` (linked to issue, branched from parent epic's branch)
>
> **Updated:**
> - Parent epic #`<parent-epic>` body: replaced `#TBD` with #`<FEAT_NUM>`
> - Blocker #`<N>` body: added `Blocks: #<FEAT_NUM>` to Dependencies
> - PI.md: replaced `#TBD` with #`<FEAT_NUM>` *(if applicable)*
> - Committed: `docs(pi): add issue number for feature <name> (#<FEAT_NUM>)` *(if applicable)*
