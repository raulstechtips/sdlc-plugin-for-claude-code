---
name: PI-1
theme: "Stabilize the core — fix bugs and complete the reference guide suite"
started: 2026-03-20
target: TBD
status: active
---

## Goals

Ensure all 8 core SDLC skills work correctly end-to-end through a full lifecycle (define PRD → plan PI → decompose into epics/features/stories → track status → reconcile → retro). Fill in the 4 missing update reference guides so that `sdlc:update` has complete coverage across all artifact levels.

## Epics

### Epic: Bug Fixes & Stabilization (#1)
**Goal:** All 8 core skills work correctly through a full end-to-end lifecycle
**Priority:** critical
**Scope seeds:**
- Rework sdlc:define to mirror superpowers brainstorming with creative freedom
- Validate and fix sdlc:init, sdlc:capture
- Validate and fix sdlc:create, sdlc:update
- Validate and fix sdlc:status, sdlc:reconcile, sdlc:retro

### Epic: Complete Update Reference Guides
**Goal:** sdlc:update has complete execution references for all 5 artifact levels
**Priority:** high
**Scope seeds:**
- Write missing update guides (pi-update.md, epic-update.md, feature-update.md, story-update.md)

## Dependencies

- Bug Fixes & Stabilization and Complete Update Reference Guides have no cross-dependencies and can run in parallel

## Worktree Strategy

- Worktree 1: Bug Fixes & Stabilization (sequential within — planning → execution → monitoring)
- Worktree 2: Complete Update Reference Guides (parallel — independent of bug fixes)
