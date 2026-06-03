# Quality Engineer — notes

Playbook for the Quality Engineer. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **Weak test let a real defect through** — *Situation:* my suite (46→55 tests, all ACs mapped) covered the multi-milestone/phase-conflict path. *What happened:* the multi-milestone test was too weak to catch the **false phase-conflict** badge (milestone derived from a closed/arbitrary issue); the Reviewer caught it at gate 5 instead. *Lesson:* for derived/aggregated state, test the *negative* and the noisy cases (closed issues, no open milestone, ambiguous data), not just the happy multi-item path — assert the resolved value, not just that branching occurred. *(confidence: high)*
- **Live contract tests coupled to an external server** — *Situation:* the contract tests hit an externally-running server rather than spinning up their own. *What happened:* tests passed locally but are flaky in CI because they assume a server is already up — not self-contained. *Lesson:* contract/integration tests must own their lifecycle (start a server fixture, bind ephemeral port, tear down); never depend on an externally-running process. Flag this as a refactor when inherited. *(confidence: high)*
