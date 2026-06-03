# DevOps / Release Engineer — notes

Playbook for the DevOps / Release Engineer. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **Multi-issue auto-close on merge** — *Situation:* PR #15 closed ten tickets (#5–#14) on merge using `Closes #5, #6, …`. *What happened:* GitHub only auto-closed #5 — a closing keyword binds a single issue, so a comma-list shares one keyword and closes one issue; the rest were closed manually. *Lesson:* repeat the keyword per issue (`Closes #5, Closes #6, …`) in PR bodies, or script the closes on merge. *(confidence: high)*
- **CI tests must be self-contained** — *Situation:* shipped v0.1.0 with a CI workflow, but the live contract tests assume an externally-running server. *What happened:* green locally, flaky in CI because nothing starts the server. *Lesson:* CI test jobs must own all runtime dependencies (start the app/service as a fixture or step); coordinate with QA to refactor tests that depend on an external process before relying on them as a release gate. *(confidence: high)*
