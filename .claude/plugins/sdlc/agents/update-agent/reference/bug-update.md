# Bug Update Reference

## Direct Update Criteria

**DIRECT** if all true:
- 1-2 fields changing (description edit, severity change, priority change, title edit, reproduction steps update, technical notes update, file scope change within same area)
- No fundamental rework of the bug's scope or root cause
- No scope expansion (bug doesn't grow to cover entirely new areas or root causes)

**ESCALATE** if any true:
- Root cause analysis reveals fundamentally different problem than originally described
- Scope expansion (bug affects entirely new areas not originally identified)
- 3+ fields changing
- Bug needs to be split into multiple bugs

Bugs do not have branches created by the update agent. Use `/sdlc:setup-dev` to create a bug branch.

## Read-Modify-Write Pattern

**The `gh issue edit --body` flag replaces the ENTIRE body.** There is no partial edit. Always use the read-modify-write pattern.

### Step 1: Read current body

Slugify the current issue title for the temp file path — lowercase, replace non-alphanumeric characters with hyphens, collapse consecutive hyphens, strip leading/trailing hyphens.

```bash
gh issue view <N> --json body --jq '.body' > /tmp/sdlc-bug-<N>-<SLUG>-body.md
```

Also read the full issue metadata for context:

```bash
gh issue view <N> --json number,title,body,labels,state
```

### Step 2: Modify the specific section

Read `/tmp/sdlc-bug-<N>-<SLUG>-body.md`, identify the section to change, and modify ONLY that section.

Common sections in a bug body:
- `## Description` — what's broken
- `## Reproduction Steps` — steps to reproduce
- `## Expected vs Actual Behavior` — what should vs does happen
- `## Affected Areas` — impacted parts of the system
- `## File Scope` — known files involved
- `## Technical Notes` — implementation considerations, root cause analysis
- `## Dependencies` — `Blocked by:` and `Blocks:` lines
- `## Parent` — PI, epic, and/or feature references (if parented)

### Step 3: Write back the full updated body

```bash
gh issue edit <N> --body-file /tmp/sdlc-bug-<N>-<SLUG>-body.md
```

### Label Changes

```bash
# Change severity
gh issue edit <N> --add-label "severity:high" --remove-label "severity:medium"

# Change priority
gh issue edit <N> --add-label "priority:high" --remove-label "priority:medium"

# Add/remove area
gh issue edit <N> --add-label "area:api"
gh issue edit <N> --remove-label "area:api"
```

### Title Changes

```bash
gh issue edit <N> --title "bug: new bug title"
```

### Clean Up

```bash
rm -f /tmp/sdlc-bug-<N>-<SLUG>-body.md
```

## Dependency Maintenance

When changing `Blocked by` or `Blocks` in the Dependencies section:

### Adding a new blocker (`Blocked by: #X`)

1. Update this bug's body: add `#X` to the `Blocked by:` line.
2. Update issue #X's body: add `#<N>` to its `Blocks:` line.

```bash
# Read the blocker's body
gh issue view <X> --json body --jq '.body' > /tmp/sdlc-bug-<N>-<SLUG>-blocker-body.md
```

Modify `/tmp/sdlc-bug-<N>-<SLUG>-blocker-body.md`:
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
gh issue edit <X> --body-file /tmp/sdlc-bug-<N>-<SLUG>-blocker-body.md
rm -f /tmp/sdlc-bug-<N>-<SLUG>-blocker-body.md
```

### Removing a blocker (`Blocked by: #X`)

1. Update this bug's body: remove `#X` from the `Blocked by:` line (if it was the only one, set to `none`).
2. Update issue #X's body: remove `#<N>` from its `Blocks:` line (if it was the only one, set to `none`).

```bash
gh issue view <X> --json body --jq '.body' > /tmp/sdlc-bug-<N>-<SLUG>-blocker-body.md
# Remove #<N> from the "Blocks:" line
gh issue edit <X> --body-file /tmp/sdlc-bug-<N>-<SLUG>-blocker-body.md
rm -f /tmp/sdlc-bug-<N>-<SLUG>-blocker-body.md
```

### Circular Dependency Check

After any dependency change, verify no circular dependency exists. Walk the blocker chain using DFS:

```bash
# Start from this bug's blockers
# For each blocker, read its body and parse "Blocked by:" line
# For each of those, do the same
# If we encounter the original bug number, there's a cycle

gh issue view <blocker> --json body --jq '.body'
# Parse "Blocked by:" line, recurse
```

If a circular dependency is detected: STOP, report the cycle to the user, and revert the dependency change.

> "Circular dependency detected: #<N> -> #<X> -> #<Y> -> #<N>. Reverting the dependency change."

## Cascade Rules

Bugs are NOT tracked in parent checklists. After updating a bug:

- **If title changed**: no parent cascade needed — bugs are not listed in parent issue checklists.
- **If deps changed**: bidirectional updates handled in Dependency Maintenance above.
- **If scope changed** (via escalation + reshape draft): no automatic cascade.
