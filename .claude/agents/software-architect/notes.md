# Software Architect — notes

Playbook for the Software Architect. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **Specify identifier shape in contracts** — *Situation:* the project registry stores `repo` as a full URL, but my architecture/interface examples used `owner/name`. *What happened:* the build had to normalize the identifier at the boundary to reconcile the two shapes. *Lesson:* when an interface consumes an existing artifact (registry, config), pin the exact format of shared identifiers in the contract and state where normalization happens, rather than leaving examples in a different shape. *(confidence: med)*
- **Derived-state needs a defined source of truth** — *Situation:* phase/milestone was a derived signal in the design (registry authoritative, conflicts flagged). *What happened:* a downstream defect surfaced — a *false* phase-conflict badge because milestone was derived from an arbitrary/closed issue rather than open issues only. *Lesson:* when a value is derived from a noisy collection (issues), specify the filter (e.g. open issues only) and the conflict-resolution rule explicitly in the contract, not just "flag conflicts." *(confidence: med)*

## hearthflix (2026-06-04)

- **Don't build an NFR on an unvalidated spike claim** — *Situation:* the Researcher's HLS spike reported that ffmpeg `delete_segments` would bound per-session disk; I adopted "disk is bounded" as an NFR and chose `-hls_playlist_type event`. *What happened:* `delete_segments` is a **no-op** under `event` playlists, so `/data` grew unbounded per session (H1, caught by the Reviewer at gate 5, not by me). I trusted an assertion that had never been demonstrated. *Lesson:* before turning a spike claim into an NFR, ask "was this **validated** or **assumed**, and under what conditions?" Treat unvalidated claims as assumptions to verify, not facts — and write the NFR so its enforcing mechanism is itself testable (a QA/Reviewer can assert the bound holds). *(confidence: high)*
- **Pair every resource NFR with an enforcement+observability seam** — *Situation:* "bounded disk" relied on an implicit ffmpeg side effect rather than an explicit teardown the system owns. *What happened:* there was no code path that *guaranteed* the bound, so nothing failed loudly when it didn't hold. *Lesson:* when I set a resource bound (disk, memory, process count), design the explicit mechanism that enforces it (e.g. session-dir teardown on stop) and make it observable/testable — don't lean on a downstream tool's incidental behavior. *(confidence: high)*
