---
slug: team-pulse-dashboard
repo: https://github.com/CKS-Equium/team-pulse-dashboard
status: active
phase: release
created: 2026-06-03
team: [orchestrator, product-manager, software-architect, ux-designer, ui-designer]
board: n/a
---

## Summary

A read-only, auto-refreshing local web dashboard that streams real GitHub activity across all
watched team projects and prominently surfaces `needs-human` items so the operator can click
straight to the issue and unblock the team. The team's first dogfood project.

## Current state

**Build complete — gate 3 passed.** Tickets #5–#14 implemented and delivered as PR #15
(10/10 unit tests pass; smoke-tested live: real API data, phaseConflict surfaced, read-only 405s).
**PAUSED at operator's request** before the Test phase, so the operator can run the app locally
first. Resume with `run-project team-pulse-dashboard` → Test (QA, gate 4) → Review → Release.
QA focus flagged by build: needs-human end-to-end (needs a labelled test issue), in-progress event
derivation, browser/UI verification.

## Decision log

- 2026-06-03 — PRD reframed from a static status snapshot to a **live activity log + "Needs you"
  panel**, per operator. Cost/run/log telemetry deferred (no data source); activity derived from
  real GitHub events. Dashboard is read-only; the operator answers `needs-human` issues in GitHub.
- 2026-06-03 — Human-in-the-loop mechanism adopted: agents raise `needs-human` issues assigned to
  the operator; an always-on scheduled Orchestrator routine (~5 min) detects answers and resumes.
  The resume logic is deferred to orchestration design (not part of the dashboard app).
- 2026-06-03 — **Gate 2 approved.** Stack = Node backend serving a static SPA; token lives only in
  the backend; server polls GitHub every 20s (ETag + back-off). Phase: registry authoritative
  (conflict flagged). `needs-human`: label-only detection. Initial log window: 24h / 100 events.
- 2026-06-03 — **Gate 3 (build) passed:** tickets #5–#14 delivered as PR #15; runs locally.
- 2026-06-03 — **Gate 4 (test) passed:** QA test plan + suite 46→55 tests, all ACs mapped.
- 2026-06-03 — **Gate 5 (review) passed after a fix loop.** Security BLOCKed on a HIGH (`javascript:`
  URL XSS sink); Reviewer flagged a false phase-conflict badge + wrong active-role. Engineer fixed
  all (safeUrl https-allowlist, CSP/headers, loopback bind, open-issue-only milestone resolution,
  role-label active-role, `status:in-progress` detection); security re-review PASS, 55/55 tests.

## Gate waivers

- _none_
