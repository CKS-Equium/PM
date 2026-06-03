---
slug: team-pulse-dashboard
repo: https://github.com/CKS-Equium/team-pulse-dashboard
status: active
phase: design
created: 2026-06-03
team: [orchestrator, product-manager, software-architect, ux-designer, ui-designer]
board: n/a
---

## Summary

A read-only, auto-refreshing local web dashboard that streams real GitHub activity across all
watched team projects and prominently surfaces `needs-human` items so the operator can click
straight to the issue and unblock the team. The team's first dogfood project.

## Current state

PRD approved (gate 1) on 2026-06-03; entering the **design** phase. Next: Software Architect
(ADRs, contracts, NFRs) + UX/UI Designers (specs) → gate 2 (human sign-off on architecture & design).

## Decision log

- 2026-06-03 — PRD reframed from a static status snapshot to a **live activity log + "Needs you"
  panel**, per operator. Cost/run/log telemetry deferred (no data source); activity derived from
  real GitHub events. Dashboard is read-only; the operator answers `needs-human` issues in GitHub.
- 2026-06-03 — Human-in-the-loop mechanism adopted: agents raise `needs-human` issues assigned to
  the operator; an always-on scheduled Orchestrator routine (~5 min) detects answers and resumes.
  The resume logic is deferred to orchestration design (not part of the dashboard app).

## Gate waivers

- _none_
