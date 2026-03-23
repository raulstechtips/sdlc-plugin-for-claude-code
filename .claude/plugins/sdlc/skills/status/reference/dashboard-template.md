# Dashboard Template

The canonical output format for the sdlc:status dashboard. Populate each section with real data. Omit sections with zero items except PI Overview and Momentum (always show).

## Section Order

1. PI Overview (always show)
2. In Progress (omit if zero)
3. Momentum (always show)
4. Blocked (omit if zero)
5. What's Next (omit if zero)
6. Parallelization (omit if zero)
7. Stale Drafts (minor callout, omit if none)

---

## PI Overview

Always show. This is the context frame for the entire dashboard.

```
## <PI title> (#N)

> <done_epics>/<total_epics> epics complete | <done_features>/<total_features> features | <done_stories>/<total_stories> stories
```

If scope filter is active, append: `(filtered: area:<name>)` or `(filtered: epic #N)`

If no active PI found:

```
## No active PI

> Run `/sdlc:define pi` to plan an increment.
```

---

## In Progress

Show items with `status:in-progress` label (stories and size:small features). Table format with parent context.

```
### In Progress (<count>)

| # | Title | Parent | Area | Age | Assignee |
|---|-------|--------|------|-----|----------|
| #N | <title> | Feature #F > Epic #E | area:X | N days | @user |
| #N | <title> | Feature #F > Epic #E | area:X | N days [STALE] | unassigned |
```

- Sort by age descending (oldest first)
- Mark `[STALE]` if age > 5 days
- Parent column shows the item's place in the hierarchy: `Feature #F > Epic #E`
- For size:small features (no parent feature), show: `Epic #E`
- Include both stories and size:small features that are in-progress

---

## Momentum

Always show. Provides velocity context before showing blockers.

```
### Momentum

- **<done_total>** / **<planned_total>** items closed this PI (<percentage>%)
- **<closed_7d>** closed in last 7 days <trend_arrow> (was <closed_prev_7d> previous week)
```

Trend arrows:
- More than previous week: arrow up
- Fewer than previous week: arrow down
- Same: flat/steady

---

## Blocked

Show items with `status:blocked` label. Trace blocker chains recursively and add hierarchy insights.

```
### Blocked (<count>)

**Root blocker: #R <root title> (<root status>)** — unblocks <X> items when resolved

- #N <title> (Feature #F > Epic #E)
  Blocked by: #B <blocker title> (<blocker status>)
  <hierarchy insight>

- #M <title> (Feature #F > Epic #E)
  Blocked by: #B2 -> #R (chain)
  <hierarchy insight>
```

- Group blocked items by their root blocker
- Show the full blocker chain when it has intermediate links: `#B -> #R`
- Hierarchy insights explain the impact in context:
  - "Last story in Feature #F — resolving this completes the feature"
  - "Blocks 2 siblings in Feature #F"
  - "Gates Feature #F completion, which gates Epic #E"

---

## What's Next

Show TODO items ranked by impact score, not just priority. This is the primary action recommendation.

```
### What's Next (<count>, ranked by impact)

1. **#N <title>** (area:X, priority:Y) — score: <S>
   Feature #F > Epic #E
   Impact: <reason>

2. **#N <title>** (area:X, priority:Y) — score: <S>
   Feature #F > Epic #E
   Impact: <reason>
```

Impact scoring components:
| Component | Points | Condition |
|-----------|--------|-----------|
| Completes feature | 4 | Last un-done item under its parent feature |
| Unblocks siblings | 3 | Appears in a sibling's `Blocked by` list |
| Gates parent completion | 2 | Completing this + done siblings completes parent, which completes grandparent |
| Priority weight | 0-1 | critical=1.0, high=0.75, medium=0.5, low=0.25 |

Score = sum of applicable components. Ties broken by priority label, then issue number ascending.

Impact reason examples:
- "Last story in Feature #F — completes the feature"
- "Unblocks #A, #B (sibling stories)"
- "Gates Feature #F which gates Epic #E"
- "High priority, no blocking impact"

---

## Parallelization

Identify independent work streams at the highest meaningful level. Adjacent to What's Next — together they form the action block.

```
### Parallelization

**Stream A: Epic #E1 — <epic title>**
- Available: #N <story>, #M <story>
- Independent: no shared blockers with Stream B

**Stream B: Epic #E2 — <epic title>**
- Available: #N <story>
- Independent: different epic, separate dependency chain
```

Within a single epic, surface feature-level parallelization:

```
**Stream A: Feature #F1 — <title> (Epic #E)**
- Available: #N <story>

**Stream B: Feature #F2 — <title> (Epic #E)**
- Available: #M <story>
- Independent: no cross-feature dependencies
```

Detection hierarchy:
1. Different epics with TODO items = epic-level streams (strongest signal)
2. Different features within same epic, no cross-deps = feature-level streams
3. Same feature, no mutual deps = story-level streams (fallback)

---

## Stale Drafts

Minor callout. Single blockquote line. Only show if stale drafts exist (>7 days old).

```
> **Stale drafts:** <filename> (<age> days), <filename> (<age> days)
```
