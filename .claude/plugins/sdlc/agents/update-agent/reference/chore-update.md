# Chore Update Reference

## Direct Update Criteria

**DIRECT** if all true:
- 1-2 fields changing (description edit, priority change, title edit, task update, technical notes update, file scope change within same area)
- No scope expansion (chore doesn't grow to cover new files/areas beyond its current scope)
- No fundamental rework of the chore's purpose
- Acceptance criteria changes limited to 1-2 items (add, remove, or reword)

**ESCALATE** if any true:
- Scope expansion (new files or areas not originally in scope)
- Fundamental rework (chore purpose or approach changes)
- AC overhaul (3+ acceptance criteria changing)
- 3+ fields changing
- Chore needs to be split

Chores do not have branches created by the update agent. Use `/sdlc:setup-dev` to create a chore branch.

## Read-Modify-Write Pattern

**The `gh issue edit --body` flag replaces the ENTIRE body.** There is no partial edit. Always use the read-modify-write pattern.

### Step 1: Read current body

Slugify the current issue title for the temp file path — lowercase, replace non-alphanumeric characters with hyphens, collapse consecutive hyphens, strip leading/trailing hyphens.

```bash
gh issue view <N> --json body --jq '.body' > /tmp/sdlc-chore-<N>-<SLUG>-body.md
```

Also read the full issue metadata for context:

```bash
gh issue view <N> --json number,title,body,labels,state
```

### Step 2: Modify the specific section

Read `/tmp/sdlc-chore-<N>-<SLUG>-body.md`, identify the section to change, and modify ONLY that section.

Common sections in a chore body:
- `## Description` — what needs doing and why
- `## Task` — the specific work
- `## Acceptance Criteria` — checkbox list
- `## File Scope` — files/directories involved
- `## Technical Notes` — implementation guidance
- `## Dependencies` — `Blocked by:` and `Blocks:` lines
- `## Parent` — PI, epic, and/or feature references (if parented)

### Step 3: Write back the full updated body

```bash
gh issue edit <N> --body-file /tmp/sdlc-chore-<N>-<SLUG>-body.md
```

### Label Changes

```bash
# Change priority
gh issue edit <N> --add-label "priority:high" --remove-label "priority:medium"

# Add/remove area
gh issue edit <N> --add-label "area:api"
gh issue edit <N> --remove-label "area:api"
```

### Title Changes

```bash
gh issue edit <N> --title "chore: new chore title"
```

### Clean Up

```bash
rm -f /tmp/sdlc-chore-<N>-<SLUG>-body.md
```

## Dependency Maintenance

When changing `Blocked by` or `Blocks` in the Dependencies section:

### Adding a new blocker (`Blocked by: #X`)

1. Update this chore's body: add `#X` to the `Blocked by:` line.
2. Update issue #X's body: add `#<N>` to its `Blocks:` line.

```bash
# Read the blocker's body
gh issue view <X> --json body --jq '.body' > /tmp/sdlc-chore-<N>-<SLUG>-blocker-body.md
```

Modify `/tmp/sdlc-chore-<N>-<SLUG>-blocker-body.md`:
- If `Blocks: none` -> replace with `Blocks: #<N>`
- If `Blocks: #A` -> change to `Blocks: #A, #<N>`
- If `Blocks: #A, #B` -> change to `Blocks: #A, #B, #<N>`
- If no Dependencies section exists -> append:
  ```
  ## Dependencies
  - Blocked by: none
  - Blocks: #<N>
  ```

```bash
gh issue edit <X> --body-file /tmp/sdlc-chore-<N>-<SLUG>-blocker-body.md
rm -f /tmp/sdlc-chore-<N>-<SLUG>-blocker-body.md
```

### Removing a blocker (`Blocked by: #X`)

1. Update this chore's body: remove `#X` from the `Blocked by:` line (if it was the only one, set to `none`).
2. Update issue #X's body: remove `#<N>` from its `Blocks:` line (if it was the only one, set to `none`).

```bash
gh issue view <X> --json body --jq '.body' > /tmp/sdlc-chore-<N>-<SLUG>-blocker-body.md
# Remove #<N> from the "Blocks:" line
gh issue edit <X> --body-file /tmp/sdlc-chore-<N>-<SLUG>-blocker-body.md
rm -f /tmp/sdlc-chore-<N>-<SLUG>-blocker-body.md
```

### Circular Dependency Check

After any dependency change, verify no circular dependency exists. Walk the blocker chain using DFS:

```bash
# Start from this chore's blockers
# For each blocker, read its body and parse "Blocked by:" line
# For each of those, do the same
# If we encounter the original chore number, there's a cycle

gh issue view <blocker> --json body --jq '.body'
# Parse "Blocked by:" line, recurse
```

If a circular dependency is detected: STOP, report the cycle to the user, and revert the dependency change.

> "Circular dependency detected: #<N> -> #<X> -> #<Y> -> #<N>. Reverting the dependency change."

## Cascade Rules

Chores are NOT tracked in parent checklists. After updating a chore:

- **If title changed**: no parent cascade needed — chores are not listed in parent issue checklists.
- **If deps changed**: bidirectional updates handled in Dependency Maintenance above.
- **If scope changed** (via escalation + reshape draft): no automatic cascade.
