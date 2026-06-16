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
| 4 | **Test → Review** | Test plan executed; acceptance criteria verified; coverage meets project threshold; no failing tests; **traceability matrix complete** — every requirement/AC traced to ≥1 test and a source location (see below) | Quality Engineer |
| 5 | **Review → Release** | Adversarial review passed (correctness + simplicity); **failure-mode pass run on any safety/data-integrity-relevant change** (see below); security review clean; no unresolved high-severity findings | Reviewer/Critic + Security Engineer |
| 6 | **Release** 👤 | CI green; build/deploy succeeds; changelog + user docs updated; rollback path known | DevOps → **human signs off** |
| 7 | **Post-mortem (closing gate)** | Self-reviews + 360 reviews complete; improvement recommendations recorded and routed (notes commits / contract PRs) | Process Engineer |

### Data-driven / authored-content projects

For projects whose behavior is **driven by authored data/content** (game data, rule tables,
config-as-code), the build and test DoD carry extra requirements — same "push correctness
upstream of review" theme as the secure-by-default baseline:

- **Gate 3 (Build):** the **full solution / all build targets** must compile each gate — not just
  the unit-testable sub-module. Authored content is **validated at load** and **fails loudly**
  (throws) on bad/unknown data rather than silently no-op'ing.
- **Gate 4 (Test):** at least one **integration test runs against the real shipped data**, not
  only hand-authored fixtures.

## Gate evidence record (every gate)

Passing a gate's DoD includes producing/updating a short **gate evidence record**
(`docs/templates/evidence-record-template.md`): a table of {exit criterion → status → evidence
(verifying command / test ref / output)} plus a **known-follow-ups register** — gaps named, not
hidden. This operationalises the **verification-is-necessary-but-insufficient** principle (see the
playtest gate below): a green build is *evidence*, not proof, so each gate leaves an honest trail
of what was checked and what is knowingly still open. Owned by the **Quality Engineer**; the
**Orchestrator** confirms it at the transition. A gate may pass with open follow-ups, but they must
be recorded and routed (ticket / notes / next gate).

The evidence record + a **one-command verifier** (`ci.sh` or equivalent) together also serve as an
**interruption-tolerance / handoff** artifact: any actor (a fresh agent, the Orchestrator, the
human) must be able to **independently re-verify and integrate** a sub-phase from its durable
artifacts alone — without resurrecting the building agent's context. This makes long autonomous runs
robust to agent/orchestrator death, not merely auditable. *(Added operator-approved 2026-06-16, from
the new-cadair A/B post-mortem — A.6's building agent died twice on transient API errors and the
sub-phase still shipped because the Orchestrator re-verified from `ci.sh` + the evidence record.)*

## Requirement → test → source traceability (gate 4)

Gate 4 is not met until a **traceability matrix** (`docs/templates/traceability-matrix-template.md`)
maps every acceptance criterion / requirement ("shall") to **≥1 test and the source location** that
implements it. An untraced requirement fails the gate. For projects with a formal spec/FS this is
**CI-enforceable** — fail the build if any matrix entry lacks a linked test or source ref. Owned by
the **Quality Engineer**; ACs must be authored traceably/testably by the **Product Manager**
upstream.

## Failure-mode pass (gate 5)

Safety/control-relevant, data-integrity, or resource-bound changes get an explicit **failure-mode
pass** at review — distinct from the happy-path correctness/scope review — that deliberately probes
the degenerate inputs (BAD/invalid input, NaN/Inf, divide-by-zero / degenerate params, boundary
values, latched/stuck states, dependency death, unbounded waits). Run by the **Reviewer/Critic**
using the named lens in its contract; reused as the same checklist wherever this class of code
appears. It complements, and does not replace, the Security Engineer's review. The same lens is run
by the **Senior Engineer as a self-review at gate 3** (authoring side), so the gate-5 reviewer pass
*confirms* rather than *discovers* the degenerate-case handling.

Where the product has a **primary operator-initiated entry action** (start batch, submit, confirm),
the **system-validation / acceptance step must exercise *that* path** (or explicitly register the
gap), not only an auto-driven one — proving the data path is necessary but insufficient if the
operator's first action goes unverified. *(Both added operator-approved 2026-06-16, from the
new-cadair A/B post-mortem; the operator-path rule recurs from colonygame — "verify the path the
human takes.")*

## Human-in-the-loop

Autonomy runs freely *between* gates. Humans are inserted only at the three high-leverage,
hard-to-reverse decisions:

1. **PRD approval** (gate 1) — are we building the right thing?
2. **Architecture approval** (gate 2) — the expensive-to-undo decision.
3. **Release** (gate 6) — the irreversible, outward-facing action.

Gates 2 and 6 (post-repo) are **raised as a `needs-human` issue on the project board** by the
Orchestrator and closed on approval, so they appear in the dashboard's "Needs you" panel. (Gate 1
is pre-repo — an interactive kickoff — so it has no board item.)

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
- **Execution mode** is a first-class alternative to these 7 gates: when the operator supplies a
  design + a gated dev plan, the project's own per-gate DoD and an adversarial reviewer-as-gate
  replace them (reviewer BLOCK = fix-before-merge, not a human stop; the final playtest is the only
  human gate). See [DESIGN.md](DESIGN.md) §7, "Execution mode."

### Playtest-gated / player-facing work

For player-facing work where the gate is a **human playtest** (game-feel), the machine-DoD is
**necessary-but-insufficient**: green `build` + green tests means "the sim didn't regress," **not**
"the game works." GUI/interaction **cannot be headless-verified** — so a recurring bug class stays
invisible to CI and only surfaces in the operator's hand, at the cost of a playtest round-trip. Two
steps gate the playtest request so the human playtest stays the *real* gate but isn't burned on
diff-visible defects:

- **GUI interaction self-audit (Senior Engineer)** — before declaring a phase playtest-ready, the
  engineer runs the player-facing build checklist (see the senior-engineer contract): controls not
  rebuilt every frame; input captured every frame, not in a fixed-timestep loop; rendered positions
  interpolated between ticks; no per-frame allocation / always-on-top-draw storms; UI enable-state
  mirrors the sim's exact preconditions; no dev/sandbox affordances in the player build.
- **Pre-playtest interaction review (Reviewer/Critic)** — before the Orchestrator raises the
  `needs-human` playtest issue, the reviewer reads the GUI/interaction diff for those same
  un-headless-verifiable pitfalls (review-only, as ever). In player-facing phases the adversarial
  *code* review thins out; this is where it pivots to earn its keep and cut the playtest iteration
  count.
