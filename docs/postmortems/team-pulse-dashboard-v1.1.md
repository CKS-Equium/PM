# Post-mortem: team-pulse-dashboard v1.1 (comment detail)

- **Project:** team-pulse-dashboard · **Iteration:** v1.1 · **Released:** v0.2.0, 2026-06-04
- **Facilitator:** Process Engineer (light retro — proportional to a small additive UI change)
- **Outcome:** Shipped. Operator-requested: activity log now shows a comment snippet + author/role (#16, PR #17).

> A deliberately **light** post-mortem: this was a 5-file, low-risk enhancement run with a
> proportionate gate set (gate 2 design consciously waived; gate 4 satisfied by automated tests;
> gate 5 review run because it touched attacker-influenceable content). Full 360 not warranted.

## What worked (the signal — mostly positive, and it validates last cycle's fixes)

1. **The upgraded contracts paid off.** This was the first run on the team *after* the hearthflix
   post-mortem promoted the **secure-by-default build checklist** into the senior-engineer contract.
   Result: the engineer applied output-encoding (`textContent`), preserved the strict CSP, and gated
   URLs up front — so **gate 5 (incl. security) passed first pass, no fix loop**, on a change that
   renders attacker-influenceable comment text. Last cycle the same class of issue *blocked* gate 5.
   Direct evidence the self-improvement loop changes outcomes.
2. **Programmatic ticket-close worked.** The other promoted fix — close tickets via `gh` after merge,
   not PR keywords — closed #16 cleanly with zero manual cleanup (vs. the manual scramble on both
   prior projects).
3. **Gate → `needs-human` (Point 1) demonstrated live.** Gate 6 raised a `needs-human` board issue
   (#18) that surfaced in the dashboard's "Needs you" panel and cleared on approval — wiring the gate
   mechanism to the dashboard's reason for being, with no dashboard code change.
4. **Proportional gating is viable.** Waiving gate 2 (logged) and running a reduced gate set for a
   small enhancement worked without loss of quality. Worth keeping as an explicit pattern.
5. **Efficiency win in the build itself:** zero new GitHub API calls (reused comment bodies the
   poller already fetched) — the engineer respected the existing rate-limit model.

## Improvement recommendations

| # | Recommendation | Target | Level | Note |
|---|----------------|--------|-------|------|
| 1 | Formalize **proportional gating** for small iterations (waive design gate w/ logged reason; fold gate-4 into tests+review) | process-engineer / gates.md | notes (done); skill-doc candidate | new pattern, proven |
| 2 | Reuse already-fetched payloads before adding API calls (rate-limit discipline) | senior-eng | notes | reinforcement |
| 3 | (cosmetic) snippet slices on UTF-16 units — a lone surrogate can be cut before the ellipsis | team-pulse-dashboard | v1.2 backlog | low, non-blocking |

## Metrics

- Gates: gate 2 **waived** (logged); gate 5 **PASS first pass** (no fix loop — improvement over both prior projects); 64/64 tests.
- Rework loops: **0**. Escaped defects: 0.
- Validated 2 applied contract/skill fixes from the hearthflix post-mortem in real use.
