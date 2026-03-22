---
name: PI-1
theme: "Stabilize the core — fix bugs and complete the reference guide suite"
started: 2026-03-20
target: TBD
status: active
---

## Goals

Ensure all core SDLC skills work correctly end-to-end through a full lifecycle (define PRD → plan PI → decompose into epics/features/stories → track status → reconcile → retro). Stabilize the execution flow by absorbing artifact creation/modification into the define skill, expanding capture to support unplanned work items, and completing the reference guide suite for the update agent.

## Epics

### Epic: Bug Fixes & Stabilization (#1)
**Goal:** All core skills work correctly through a full end-to-end lifecycle
**Priority:** critical
**Scope seeds:**
- Rework sdlc:define to mirror superpowers brainstorming with creative freedom
- Absorb execution into define, remove standalone create/update skills
- Expand sdlc:capture to support bug, chore, and triage types
- Validate and fix sdlc:init
- Validate and fix sdlc:status, sdlc:reconcile, sdlc:retro

### Epic: Complete Update Reference Guides
**Goal:** Update agent has complete execution references for all 5 artifact levels
**Priority:** high
**Scope seeds:**
- Review and fix existing update guides (pi-update.md, epic-update.md, feature-update.md, story-update.md, prd-update.md)

## Dependencies

- Bug Fixes & Stabilization and Complete Update Reference Guides have no cross-dependencies and can run in parallel

## Worktree Strategy

- Worktree 1: Bug Fixes & Stabilization (sequential within — planning → execution → monitoring)
- Worktree 2: Complete Update Reference Guides (parallel — independent of bug fixes)
