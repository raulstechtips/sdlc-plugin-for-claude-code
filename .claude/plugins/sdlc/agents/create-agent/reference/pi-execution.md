# PI Execution Reference

## Required Fields

### Frontmatter
- `name` — PI identifier (e.g., `PI-1`, `PI-2`)
- `theme` — one-line theme description
- `priority` — one of: `critical`, `high`, `medium`, `low`
- `areas` — array of area labels (e.g., `[auth, api]`)

### Body Sections
- `## Goals` — non-empty
- `## Timeline` — non-empty
- `## Epics` — at least one `### Epic: <name> (#TBD)` subsection

Additional sections (Dependency Graph, Worktree Strategy) are expected but not blocking.

## Execution Steps

### 1. Check for Active PI

```bash
ACTIVE_PI=$(gh issue list --label "type:pi" --state open --json number,title --jq '.[0]')
```

If `ACTIVE_PI` is non-empty, proceed to step 2. If empty, skip to step 3.

### 2. Close Active PI (if exists)

```bash
gh issue close <ACTIVE_PI_NUM>
```

**Note:** Decision log baking into the PRD is deferred — it is not part of this flow.

### 3. Create the PI Issue

Strip the YAML frontmatter from the draft and write the body to a temp file, then create the issue:

```bash
# Strip frontmatter from draft, write body to temp file
# (everything after the closing --- of the frontmatter)

cat <<'BODY' > /tmp/sdlc-pi-body.md
<draft body content without frontmatter>
BODY

PI_URL=$(gh issue create \
  --title "<name>" \
  --body-file /tmp/sdlc-pi-body.md \
  --label "type:pi" \
  --label "priority:<priority>" \
  --label "area:<area1>" \
  --label "area:<area2>")
PI_NUM=$(echo "$PI_URL" | grep -o '[0-9]*$')
```

**Note:** Do NOT add a `status:` label to PI issues. Status labels are for stories only.

### 4. Create and Link Branch

Follow [`branch-creation.md`](branch-creation.md) with:
- `ISSUE_NUM` = `<PI_NUM>`
- `ISSUE_TITLE` = `<name>`
- `LEVEL` = `pi`
- `PARENT_ISSUE` = `none`

### 5. Create Epic Stubs

For each `### Epic: <name> (#TBD)` subsection in the PI draft's `## Epics` section, create a stub issue:

```bash
cat <<'BODY' > /tmp/sdlc-epic-stub-body.md
## Overview

**Goal:** <goal from PI epic subsection>

**Scope seeds:**
<scope seeds from PI epic subsection>

## Parent

PI: #<PI_NUM>
BODY

EPIC_URL=$(gh issue create \
  --title "<epic name>" \
  --body-file /tmp/sdlc-epic-stub-body.md \
  --label "type:epic" \
  --label "priority:<priority from PI epic subsection>")
EPIC_NUM=$(echo "$EPIC_URL" | grep -o '[0-9]*$')
```

**Note:** No branch is created for stubs. No status label.

### 6. Backfill PI Body

For each created epic stub, replace the `(#TBD)` placeholder in the PI issue body with the real epic number:

```bash
PI_BODY=$(gh issue view <PI_NUM> --json body --jq '.body')
UPDATED_BODY=$(echo "$PI_BODY" | sed "s/### Epic: <EPIC_NAME> (#TBD)/### Epic: <EPIC_NAME> (#<EPIC_NUM>)/")
gh issue edit <PI_NUM> --body "$UPDATED_BODY"
```

Repeat for each epic stub created in step 5.

### 7. Clean Up Temp Files

```bash
rm -f /tmp/sdlc-pi-body.md /tmp/sdlc-epic-stub-body.md
```

## Report Format

> **Created:**
> - PI: #`<PI_NUM>` — "`<name>`"
>   - Labels: `type:pi`, `priority:<priority>`, `area:<areas>`
>   - Branch: `pi/<PI_NUM>-<slugified-name>` (linked to issue)
> - Epic stubs:
>   - #`<EPIC1_NUM>` — "`<epic1 name>`"
>   - #`<EPIC2_NUM>` — "`<epic2 name>`"
>
> **Updated:**
> - PI #`<PI_NUM>` body: replaced `#TBD` with real epic numbers
>
> **Closed:** (if applicable)
> - PI #`<OLD_PI_NUM>` — "`<old PI name>`"
