# Product Manager — notes

Playbook for the Product Manager. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **PRD authored from a captured transcript** — *Situation:* gate-1 discovery; I was asked to draft the PRD but could not run the live interview myself (a subagent has no interactive channel to the operator). *What happened:* the Orchestrator ran the live interview in the main thread and handed me the Q&A transcript, from which I drafted the PRD. *Lesson:* my discovery input is the Orchestrator's captured interview transcript, not a live conversation — design the PRD flow around consuming that artifact. *(confidence: high)*
- **First framing was too static** — *Situation:* initial PRD framed the dashboard as a static status snapshot. *What happened:* gate 1 took 2 rounds; the operator reframed it to a **live activity log + "Needs-you" panel** (surface `needs-human` items, click straight to the issue). *Lesson:* for monitoring/ops tools, probe early for "what action does this drive?" — favour live, action-oriented framings over static snapshots; ask the action question in round 1. *(confidence: med)*
