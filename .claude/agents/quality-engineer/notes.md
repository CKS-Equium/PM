# Quality Engineer — notes

Playbook for the Quality Engineer. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **Weak test let a real defect through** — *Situation:* my suite (46→55 tests, all ACs mapped) covered the multi-milestone/phase-conflict path. *What happened:* the multi-milestone test was too weak to catch the **false phase-conflict** badge (milestone derived from a closed/arbitrary issue); the Reviewer caught it at gate 5 instead. *Lesson:* for derived/aggregated state, test the *negative* and the noisy cases (closed issues, no open milestone, ambiguous data), not just the happy multi-item path — assert the resolved value, not just that branching occurred. *(confidence: high)*
- **Live contract tests coupled to an external server** — *Situation:* the contract tests hit an externally-running server rather than spinning up their own. *What happened:* tests passed locally but are flaky in CI because they assume a server is already up — not self-contained. *Lesson:* contract/integration tests must own their lifecycle (start a server fixture, bind ephemeral port, tear down); never depend on an externally-running process. Flag this as a refactor when inherited. *(confidence: high)*

## hearthflix (2026-06-04)

- **Test the resource-bound NFR, not just the feature** — *Situation:* I grew the suite to 49/49 at gate 4 with good functional coverage (added transcode ready-signal both branches, capacity reaping, watchdog, path-traversal). *What happened:* I tested that transcoding worked, but I did **not** assert the design's "disk is bounded per session" NFR — so the unbounded-`/data` bug (H1) slid past gate 4 to the Reviewer at gate 5. *Lesson:* when the design states a resource bound (disk/memory/process count), write a test that **asserts the bound holds** (e.g. session dir is empty/removed after stop), not just that the happy-path feature runs. Treat every NFR as a test obligation. *(confidence: high)*
- **Cover dependency-death and unbounded-wait paths** — *Situation:* the transcode entered a wait-for-ready state. *What happened:* "Preparing forever" when ffmpeg died (M1) was a wait-state defect; my added tests covered the ready signal but the death-during-wait case needed explicit coverage. *Lesson:* for any wait/poll state, add a negative test where the dependency dies/fails immediately, and assert the system reaches an error state within a bound — don't only test the success transition. *(confidence: high)*
