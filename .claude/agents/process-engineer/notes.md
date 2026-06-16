# Process Engineer — notes

Playbook for the Process Engineer. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## new-cadair A/B experiment (2026-06-16) — autonomous Phase-A rebuild vs reference

- **A one-command verifier + a self-describing evidence record per sub-phase is a resilience/handoff artifact, not just a CI nicety** — *Situation:* the autonomous build of new-cadair Phase A (six sub-phases, branch+PR each, self-merged on green DoD), benchmarked against OG-cadair. *What happened:* on the final sub-phase (A.6) the building agent **died twice on transient API errors** (500, then 529) with no final report. The work survived and shipped anyway because each sub-phase had left (a) an incremental commit on its branch, (b) an honest **evidence record**, and (c) a **one-command `ci.sh`**. The orchestrator re-verified the half-built A.6 by *running `ci.sh` directly* (7/7 green, 158 tests, 5× deterministic) and *reading the evidence doc for honesty*, then integrated it — **without resurrecting the dead agent's context.** *Lesson:* require every build sub-phase to leave a **one-command verifier + a self-describing evidence record**, and justify it explicitly as an **interruption-tolerance / handoff** property — any actor (a fresh agent, the orchestrator, the human) must be able to independently verify and integrate a sub-phase from its durable artifacts alone. This makes long autonomous runs robust to orchestrator/agent death, not just auditable. **Candidate gate-DoD addition.** *(confidence: high)*
