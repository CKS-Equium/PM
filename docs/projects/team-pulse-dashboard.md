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

**v1.1 SHIPPED — v0.2.0, 2026-06-04.** Activity log now shows a comment snippet + author/role
(#16, PR #17). Gate 5 passed **first pass** (the upgraded secure-by-default contract worked — no
fix loop on attacker-influenceable comment text); 64/64 tests; zero new API calls; #16 closed
programmatically; gate 6 surfaced as a `needs-human` board item (#18) per the new convention.
Light post-mortem: [docs/postmortems/team-pulse-dashboard-v1.1.md](../postmortems/team-pulse-dashboard-v1.1.md).

> **v0.1.0 (shipped 2026-06-03):** all 7 gates passed; PR #15; 55/55 tests; CI; runs locally.
> Post-mortem: [docs/postmortems/team-pulse-dashboard.md](../postmortems/team-pulse-dashboard.md).

> **v0.1.0 (shipped 2026-06-03):** all 7 gates passed; PR #15; 55/55 tests; CI; runs locally.
> Post-mortem: [docs/postmortems/team-pulse-dashboard.md](../postmortems/team-pulse-dashboard.md).

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
- 2026-06-04 — **v1.1 shipped (v0.2.0).** Comment-detail enhancement (#16, PR #17). Gate 2 waived
  (proportional); gate 5 PASS first pass (upgraded secure-by-default contract validated — no fix
  loop); 64/64; programmatic close; gate 6 raised `needs-human` #18 (Point 1 demoed). Light gate-7
  retro confirms the applied contract/skill fixes work in practice.

## Gate waivers

- 2026-06-04 (v1.1) — **Gate 2 (design) waived.** The comment-detail enhancement is a small,
  localized change within the existing architecture (aggregator fetches comment bodies; activity
  log renders a snippet + author). No new ADR needed; the engineer documents the rate-limit
  handling in the PR and the Reviewer checks it. Build → test → review → release as normal.
