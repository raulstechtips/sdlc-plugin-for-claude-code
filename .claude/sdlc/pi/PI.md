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

### Epic: Bug Fixes & Stabilization (#TBD)
- Feature: Planning Skills Stabilization (#TBD) — validate and fix sdlc:init, sdlc:define, sdlc:capture
- Feature: Execution Skills Stabilization (#TBD) — validate and fix sdlc:create, sdlc:update
- Feature: Monitoring Skills Stabilization (#TBD) — validate and fix sdlc:status, sdlc:reconcile, sdlc:retro

### Epic: Complete Update Reference Guides (#TBD)
- Feature: Write Missing Update Guides (#TBD) — create pi-update.md, epic-update.md, feature-update.md, story-update.md

## Dependency Graph

- Epic "Bug Fixes & Stabilization" and Epic "Complete Update Reference Guides" have no cross-dependencies and can run in parallel
- Within Bug Fixes, Planning Skills should be validated first since Execution and Monitoring skills depend on artifacts produced by planning skills
- The update guides epic has no internal ordering constraints — the 4 guides can be written in any order

## Worktree Strategy

- Worktree 1: Bug Fixes & Stabilization (sequential within — planning → execution → monitoring)
- Worktree 2: Complete Update Reference Guides (parallel — independent of bug fixes)
