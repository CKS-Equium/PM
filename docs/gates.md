# Phase Gates & Definition of Done

Owned by the **Process Engineer**. A phase may not start until the prior gate's Definition of
Done (DoD) is met. The **Orchestrator** enforces gates; it may consciously *waive* one only by
logging the reason in the project's registry entry.

Legend: 👤 = **human approval required** before passing.

## Lifecycle gates

| # | Gate (exit of phase) | Definition of Done | Owner verifies |
|---|---|---|---|
| 1 | **Discovery → Design** 👤 | PRD exists with user problem, scope, and **testable** acceptance criteria; open questions resolved or logged | Product Manager → **human signs off** |
| 2 | **Design → Build** 👤 | Architecture documented as ADRs; interface contracts + NFRs defined; UX flows and UI specs approved; plan broken into tickets with dependencies | Architect + Design → **human signs off** |
| 3 | **Build → Test** | All planned tickets implemented; code compiles/builds; self-review done; no `TODO`/stub in delivered scope; **secure-by-default baseline met** (security headers/CSP, dependency CVE audit, non-root container, no unbounded wait-states — see the senior-engineer contract) | Senior Engineer |
| 4 | **Test → Review** | Test plan executed; acceptance criteria verified; coverage meets project threshold; no failing tests | Quality Engineer |
| 5 | **Review → Release** | Adversarial review passed (correctness + simplicity); security review clean; no unresolved high-severity findings | Reviewer/Critic + Security Engineer |
| 6 | **Release** 👤 | CI green; build/deploy succeeds; changelog + user docs updated; rollback path known | DevOps → **human signs off** |
| 7 | **Post-mortem (closing gate)** | Self-reviews + 360 reviews complete; improvement recommendations recorded and routed (notes commits / contract PRs) | Process Engineer |

## Human-in-the-loop

Autonomy runs freely *between* gates. Humans are inserted only at the three high-leverage,
hard-to-reverse decisions:

1. **PRD approval** (gate 1) — are we building the right thing?
2. **Architecture approval** (gate 2) — the expensive-to-undo decision.
3. **Release** (gate 6) — the irreversible, outward-facing action.

### Async escalation (`needs-human`)

Outside the three gates, any agent that needs a human decision opens a **GitHub issue labelled
`needs-human`** in the project repo, assigned to the operator, and the work waits. The operator
answers **in the issue**; an always-on scheduled Orchestrator routine (~5 min) detects the answer
and resumes the agent. This keeps escalation **durable and asynchronous** (it survives session
restarts) rather than blocking on a live prompt. The dashboard's "Needs you" panel surfaces these.

*Status: the convention is adopted; the scheduled-resume routine is an orchestration build still to
come (a `schedule`-skill cron job) — until then, escalations are checked manually.*

## Gate discipline

- **Never** pass a gate with its DoD unmet unless explicitly waived (with logged reason).
- A failed gate returns work to the **owning** phase, not further back, unless the failure is rooted
  upstream (e.g. a Review finding that traces to a bad requirement → back to Discovery).
- Gate outcomes are signals for the post-mortem and the self-improvement loop (see
  [DESIGN.md](DESIGN.md) §5, §7).
