---
slug: team-pulse-dashboard
repo: https://github.com/CKS-Equium/team-pulse-dashboard
status: shipped
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

**SHIPPED & CLOSED — v0.1.0, 2026-06-03.** All 7 gates passed; PR #15 merged to `main`, tagged
[v0.1.0](https://github.com/CKS-Equium/team-pulse-dashboard/releases/tag/v0.1.0). 55/55 tests; CI
in place; runs locally (`npm install && npm start` → http://localhost:3000). **Post-mortem (gate 7)
recorded:** [docs/postmortems/team-pulse-dashboard.md](../postmortems/team-pulse-dashboard.md) — 10
recommendations, lessons written to 7 agents' notes.md; skill/contract follow-ups recorded.

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
- 2026-06-03 — **Gate 6 (release) approved & shipped v0.1.0.** PR #15 merged to main; release tagged.
  GitHub `Closes #5, #6, …` only auto-closed #5 (single-keyword-comma-list gotcha) — rest closed manually.
- 2026-06-03 — **Gate 7 (post-mortem) recorded — project CLOSED.** Self + 360 reviews; lessons → 7
  agents' notes.md; 10 recommendations (skill fixes to start-project, QA self-contained tests, a
  security-checklist contract candidate). Gates first-try 5/7; 0 defects escaped to v0.1.0.

## Gate waivers

- _none_
