# Product Manager — notes

Playbook for the Product Manager. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **PRD authored from a captured transcript** — *Situation:* gate-1 discovery; I was asked to draft the PRD but could not run the live interview myself (a subagent has no interactive channel to the operator). *What happened:* the Orchestrator ran the live interview in the main thread and handed me the Q&A transcript, from which I drafted the PRD. *Lesson:* my discovery input is the Orchestrator's captured interview transcript, not a live conversation — design the PRD flow around consuming that artifact. *(confidence: high)*
- **First framing was too static** — *Situation:* initial PRD framed the dashboard as a static status snapshot. *What happened:* gate 1 took 2 rounds; the operator reframed it to a **live activity log + "Needs-you" panel** (surface `needs-human` items, click straight to the issue). *Lesson:* for monitoring/ops tools, probe early for "what action does this drive?" — favour live, action-oriented framings over static snapshots; ask the action question in round 1. *(confidence: med)*

## hearthflix (2026-06-04)

- **Transcript-driven PRD cleared gate 1 first try** — *Situation:* second project; I drafted the PRD from the Orchestrator's captured discovery transcript under the improved start-project flow. *What happened:* gate 1 passed **first try** with all 13 ACs testable and the no-auth/trusted-LAN risk explicitly surfaced for the operator to accept. *Lesson:* the transcript-consumption pattern works — the keys were testable ACs and surfacing the security/scope trade-off (no-auth) as an explicit operator decision rather than an assumption. Keep making risk acceptances explicit in the PRD. *(confidence: med)*

## new-cadair A/B experiment (2026-06-16)

- **Acceptance must name the primary OPERATOR-INITIATED path, or it silently won't be exercised** — *Situation:* the autonomous A/B build of Cadair Phase A; new-cadair auto-runs its recipe on boot, OG requires an operator "Start Batch" press to run. *What happened:* the runnable demo + validation gate proved the live data path end-to-end, but because the recipe auto-ran, the **operator-initiated start path was never exercised** by any gate — a real operator's first action went unverified. *Lesson:* when a product has a **primary operator entry action** (start batch, submit, confirm), the PRD's acceptance criteria must make that action an explicit, testable AC — otherwise an auto-driven demo/gate proves the machine path and leaves the human's actual first step unverified. Write ACs around the **operator's entry action**, not just the resulting data flow. *(confidence: med)*
