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

Strip the YAML frontmatter from the draft and write the body to a temp file, then create the issue.

Slugify the PI name for the temp file path — lowercase, replace non-alphanumeric characters with hyphens, collapse consecutive hyphens, strip leading/trailing hyphens.

```bash
# Strip frontmatter from draft, write body to temp file
# (everything after the closing --- of the frontmatter)
# SLUG = slugified <name> (e.g., "PI-2: Core Platform" -> "pi-2-core-platform")

cat <<'BODY' > /tmp/sdlc-pi-<SLUG>-body.md
<draft body content without frontmatter>
BODY

PI_URL=$(gh issue create \
  --title "<name>" \
  --body-file /tmp/sdlc-pi-<SLUG>-body.md \
  --label "type:pi" \
  --label "priority:<priority>" \
  --label "area:<area1>" \
  --label "area:<area2>")
PI_NUM=$(echo "$PI_URL" | grep -o '[0-9]*$')

rm -f /tmp/sdlc-pi-<SLUG>-body.md
```

**Note:** Do NOT add a `status:` label to PI issues. Status labels are for stories only.

### 4. Create and Link Branch

Follow [`branch-creation.md`](branch-creation.md) with:
- `ISSUE_NUM` = `<PI_NUM>`
- `ISSUE_TITLE` = `<name>`
- `LEVEL` = `pi`
- `PARENT_ISSUE` = `none`

## Report Format

> **Created:**
> - PI: #`<PI_NUM>` — "`<name>`"
>   - Labels: `type:pi`, `priority:<priority>`, `area:<areas>`
>   - Branch: `pi/<PI_NUM>-<slugified-name>` (linked to issue)
>
> **Closed:** (if applicable)
> - PI #`<OLD_PI_NUM>` — "`<old PI name>`"
