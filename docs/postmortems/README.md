# Post-mortems

After a project is released, the **Process Engineer** facilitates a post-mortem — the closing
gate (a project isn't "done" until this exists). It is the primary feedback signal for the team's
recursive self-improvement (see [DESIGN.md](../DESIGN.md) §5, §7).

One report per project: `docs/postmortems/<slug>.md`. Template: [`_template.md`](_template.md).

## Two stages

1. **Self-review** — each agent assesses its own performance (what it kept, what it would drop).
   Lessons flow into that agent's own `notes.md` (the unrestricted self-edit path).
2. **360 review** — each agent reviews the collaborators **up and down its org chart that it
   actually worked with** (not all-to-all). Feedback is aggregated into improvement
   recommendations.

## Routing recommendations

- **Notes-level** (a heuristic, a gotcha) → appended to the target agent's `notes.md`.
- **Contract-level** (a scope change) → a **review-gated PR** against the target's `persona.md`,
  curated by the Process Engineer.

The report's **Actions taken** section links each recommendation to the actual notes commit or
contract PR — so it can later be audited whether the team acted on what it learned.

## Reports

| Project | Date | Recommendations | Contract changes |
|---------|------|-----------------|------------------|
| [team-pulse-dashboard](team-pulse-dashboard.md) | 2026-06-03 | 10 (7 notes applied; 3 skill + 1 settings + 1 contract-candidate routed) | 0 applied (1 candidate: security-engineer data-driven-UI checklist, pending 2nd-project proof) |
| [hearthflix](hearthflix.md) | 2026-06-04 | 9 (7 notes applied; 2 contract-candidates + 1 skill fix + 1 process decision routed; 2 RECURRING) | 0 applied (2 candidates routed: secure-by-default build-DoD/senior-eng [RECURRING, now proven 2×], researcher validate-don't-assert) |
| [team-pulse-dashboard v1.1](team-pulse-dashboard-v1.1.md) | 2026-06-04 | 3 (light retro) | n/a — **validated** the prior cycle's applied fixes (secure-by-default + programmatic close) in practice |
