# Software Architect — notes

Playbook for the Software Architect. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **Specify identifier shape in contracts** — *Situation:* the project registry stores `repo` as a full URL, but my architecture/interface examples used `owner/name`. *What happened:* the build had to normalize the identifier at the boundary to reconcile the two shapes. *Lesson:* when an interface consumes an existing artifact (registry, config), pin the exact format of shared identifiers in the contract and state where normalization happens, rather than leaving examples in a different shape. *(confidence: med)*
- **Derived-state needs a defined source of truth** — *Situation:* phase/milestone was a derived signal in the design (registry authoritative, conflicts flagged). *What happened:* a downstream defect surfaced — a *false* phase-conflict badge because milestone was derived from an arbitrary/closed issue rather than open issues only. *Lesson:* when a value is derived from a noisy collection (issues), specify the filter (e.g. open issues only) and the conflict-resolution rule explicitly in the contract, not just "flag conflicts." *(confidence: med)*
