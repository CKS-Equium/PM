# Post-mortem: hearthflix

- **Project:** hearthflix · **Repo:** https://github.com/CKS-Equium/hearthflix
- **Dates:** 2026-06-04 → 2026-06-04 (v0.1.0)
- **Facilitator:** Process Engineer
- **Outcome:** Shipped (v0.1.0) — the team's **second** project

> A self-hosted, YouTube-style web app for an Arch Linux server: browses an on-disk media library
> as a thumbnail grid and plays any video full-screen in the browser via pass-through or on-demand
> ffmpeg/HLS transcoding, on a trusted LAN with no auth, deployed via Docker with a read-only
> mounted media volume (photos shipped as a v1 extra). Participating roles: orchestrator,
> product-manager, **researcher-analyst (first use this project)**, software-architect, ux-designer,
> ui-designer, project-manager, senior-software-engineer, quality-engineer, reviewer-critic,
> security-engineer. No separate devops/tech-writer phase — the build engineer wrote the README and
> the operator did the Docker deploy. The junior engineer was not separately spawned.

> **Reading note:** lessons marked **RECURRING** were also raised in the team-pulse-dashboard
> post-mortem and recurred here — they are the strongest candidates for contract/skill promotion.

## 1. Self-assessments

### Orchestrator
- **Kept:** Ran the improved start-project flow — live interview in the main thread, **gate-1
  before repo creation**, project dir auto-registered. Gate 1 passed first try with no orphan repo
  and **zero directory-access prompts**. Enforced gates and drove the gate-5 BLOCK → fix → PASS loop.
- **Dropped / would change:** Multi-issue auto-close on merge **still** failed (only ~6/13 closed)
  even with the team-pulse "one keyword per issue" fix; closed the rest manually. No-Bash agents
  (PM, researcher, UX/UI) still can't self-narrate, so I narrated for them.
- **Lesson → notes.md:** Our prior post-mortem fixes paid off (verify them each run); close tickets
  **programmatically in a loop** after merge, not via PR keywords (supersedes team-pulse rec #4);
  resolve the no-Bash narration gap. *(confidence: high)*

### Product Manager
- **Kept:** Drafted the PRD from the captured transcript; **gate 1 passed first try** with 13
  testable ACs and the no-auth/trusted-LAN risk surfaced as an explicit operator acceptance.
- **Dropped / would change:** Nothing structural this run.
- **Lesson → notes.md:** Transcript-driven PRD pattern works; keep making risk/scope trade-offs
  (no-auth) explicit operator decisions in the PRD. *(confidence: med)*

### Researcher / Analyst *(first outing)*
- **Kept:** The HLS spike gave the Architect a concrete, buildable streaming approach
  (HLS + hls.js over progressive), de-risking the core unknown of the project.
- **Dropped / would change:** My spike **asserted** that ffmpeg `delete_segments` would bound
  per-session disk — a claim I never empirically validated. Under `-hls_playlist_type event` it is
  a **no-op**. That assertion became a false "bounded disk" NFR and surfaced as the H1
  unbounded-`/data` blocker at gate 5.
- **Lesson → notes.md:** A spike must **validate with evidence, not assert**; any claim that will
  become an NFR must be demonstrated, labelled validated-vs-assumed, and state the conditions
  (which ffmpeg mode) it holds under. *(confidence: high)*

### Software Architect
- **Kept:** Gate 2 approved **first try** — HLS + hls.js, FastAPI/Python/SQLite, ffprobe
  pass-through (+remux tier), eager thumbnails, Docker (RO `/media`, RW `/data`), per-session
  ffmpeg with 90s watchdog + cap 2, VAAPI opt-in seam, photos folded in cheaply.
- **Dropped / would change:** Adopted the Researcher's unvalidated `delete_segments` claim as a
  "bounded disk" NFR; the bound had no system-owned enforcement mechanism, so nothing failed loudly
  when it didn't hold (H1).
- **Lesson → notes.md:** Don't build an NFR on an unvalidated spike claim (validated vs assumed?);
  pair every resource bound with an explicit, testable enforcement seam rather than a tool's
  incidental behavior. *(confidence: high)*

### Senior Software Engineer
- **Kept:** Delivered all 13 tickets (#4–#16) as PR #17 with 31 tests and a live smoke test
  (scan/thumbnail/pass-through/HLS lifecycle/cap-503/clean shutdown); turned the gate-5 blockers
  around to a clean re-review fast.
- **Dropped / would change:** **RECURRING** — shipped without security headers/CSP (FIND-01),
  CVE-bearing deps (FIND-02), a root container (FIND-03) and `0.0.0.0` bind (FIND-04), all
  first-write defects despite my team-pulse "secure-by-default at first write" note. Also: relied
  on ffmpeg's `delete_segments` to bound disk (H1) and let `ready()` ignore `process.poll()` so a
  dead ffmpeg meant "Preparing forever" (M1).
- **Lesson → notes.md:** Run a fixed **secure-by-default build checklist before review** (headers/
  CSP, dependency audit, non-root container, loopback default); no wait-state may be unbounded —
  poll dependency liveness; own teardown of resources I create. *(confidence: high)*

### Quality Engineer
- **Kept:** Grew the suite to 59/59, adding negative/edge tests the Reviewer would otherwise need —
  transcode ready-signal (both branches), capacity reaping, watchdog, **path-traversal rejection**,
  cap behavior. Gate 4 passed cleanly.
- **Dropped / would change:** Tested that transcoding *worked* but did **not** assert the "disk is
  bounded per session" NFR, so H1 slid past gate 4 to the Reviewer at gate 5; the
  death-during-wait case (M1) also needed explicit coverage.
- **Lesson → notes.md:** Treat every resource-bound NFR as a test obligation (assert the bound, not
  just the feature); add negative tests where a waited-on dependency dies immediately. *(confidence: high)*

### Reviewer / Critic
- **Kept:** **Earned its keep again** — BLOCKed gate 5 on real, release-blocking defects tests
  missed: H1 unbounded disk (false NFR), M1 "Preparing forever", M2 sparse-keyframe remux stall.
  Re-verified PASS after the fix loop. Performed to contract; feedback routed as 360 input to
  Researcher/Architect/Engineering/QA, no durable notes lesson this run.

### Security Engineer
- **Kept:** The data-driven-UI checklist (promoted into the contract after team-pulse) caught the
  predictable issues; path-traversal/subprocess/output-encoding/read-only all PASSED. Re-reviewed
  to PASS after fixes; kept blocking on HIGH.
- **Dropped / would change:** **RECURRING** — the same *class* of basics (headers/CSP, CVE deps,
  root container, open bind) reached review as first-write defects, as at team-pulse. I'm the
  detector when the Engineer should be.
- **Lesson → notes.md:** Push a build-time secure-by-default checklist upstream to the Engineer so
  I'm the safety net; extend my standard pass beyond app-layer XSS/CSP to container (non-root) and
  dependency-CVE hygiene. *(confidence: high)*

### Project Manager
- **Kept:** Clean 13-ticket plan (#4–#16) with dependencies; build tracked cleanly to delivery as
  PR #17. No durable notes-level lesson this run.

### UX Designer / UI Designer
- **Kept:** Grid + full-screen watch-page flows and visual specs cleared gate 2 first try and
  needed no rework. No durable notes-level lesson this run.

## 2. 360 reviews (chain neighbors)

### Orchestrator → Researcher / Analyst
- **Worked well:** First spike de-risked the core streaming unknown and gave the Architect a
  decisive direction.
- **Friction / suggestion:** The disk-bound claim was asserted, not validated, and propagated into
  a false NFR — qualify spike findings as validated-vs-assumed with their conditions.

### Researcher / Analyst → Software Architect
- **Worked well:** The Architect took the spike and produced a buildable gate-2 design first try.
- **Friction / suggestion:** A spike claim shouldn't be adopted as an NFR without a validation
  step on either side — happy to label confidence, but the NFR also needs a system-owned
  enforcement mechanism.

### Software Architect → Researcher / Analyst
- **Worked well:** Clear, actionable streaming recommendation.
- **Friction / suggestion:** Distinguish "I validated X under conditions Y" from "X is the
  expected behavior" — I treated the disk-bound claim as fact and it became H1.

### Software Architect → Senior Software Engineer
- **Worked well:** Faithful 13-ticket implementation; fast, complete fix turnaround at gate 5.
- **Friction / suggestion:** Disk teardown, process-liveness on waits, and secure-by-default were
  first-write gaps, not architecture gaps — apply them proactively.

### Senior Software Engineer → Software Architect
- **Worked well:** Decisive stack and tiering (pass-through/remux/transcode, watchdog, cap).
- **Friction / suggestion:** The "bounded disk" NFR rested on a tool side effect with no enforcing
  code path; specify the explicit teardown mechanism so the bound is something I can build to and
  QA can assert.

### Quality Engineer → Senior Software Engineer
- **Worked well:** Code was testable; I could add path-traversal, capacity and watchdog coverage.
- **Friction / suggestion:** Wire NFRs (disk bound) to observable hooks so they're assertable, and
  surface dependency-death as an explicit error state for negative tests.

### Reviewer/Critic → Quality Engineer
- **Worked well:** Strong, growing AC + edge coverage (59/59); path-traversal caught at test time.
- **Friction / suggestion:** Resource-bound NFRs (disk) and dependency-death waits weren't
  asserted — those reached me as H1/M1 instead of failing at gate 4.

### Security Engineer → Senior Software Engineer
- **Worked well:** Remediated all four findings correctly and fast; path-traversal/read-only were
  right at first write.
- **Friction / suggestion:** Headers/CSP, dependency pinning, non-root container and loopback bind
  are predictable for any deployed web service — apply them at first write (recurring across two
  projects).

### Reviewer/Critic + Security Engineer → Orchestrator
- **Worked well:** Gate 5 was respected — the BLOCK paused release, the fix loop ran back through
  build to a clean re-review, and gate 6 was a real operator acceptance.
- **Friction / suggestion:** None on gate discipline; the gate is doing its job (two projects).

### Orchestrator → Product Manager
- **Worked well:** Transcript-driven PRD cleared gate 1 first try with testable ACs and an explicit
  no-auth risk acceptance.
- **Friction / suggestion:** None this run.

## 3. Improvement recommendations

| # | Recommendation | Target role | Level (notes / contract / skill) | Severity | Owner |
|---|----------------|-------------|----------------------------------|----------|-------|
| 1 | **Spikes must validate empirically, not assert** — any spike claim that will become an NFR or load-bearing assumption must be demonstrated (repro/measurement), labelled validated-vs-assumed, with the conditions it holds under stated. **Contract candidate** (first project for this role — codify the standard early). | researcher-analyst | notes (done); **candidate contract item** | high | Researcher |
| 2 | **Secure-by-default build checklist before review** — security headers + strict CSP, dependency CVE audit/pin, non-root container, configurable bind defaulting to loopback. **RECURRING** (team-pulse rec #7/#8). Now proven necessary across **two** projects → promote into the senior-engineer contract and/or the **gate-3 build DoD**. | senior-software-engineer (+ gates.md) | notes (done); **candidate contract / build-DoD** | high | Senior Eng |
| 3 | **Close tickets programmatically in a loop after merge** — do not rely on PR-body `Closes #N` keywords at all (only ~6/13 closed). **Supersedes team-pulse rec #4.** Fix in start-project/run-project and/or smart-merge to iterate `gh issue close` over the ticket list. | orchestrator / devops-release-engineer | notes (done); **skill fix** | med | Orchestrator |
| 4 | **Don't adopt an unvalidated claim as an NFR; pair every resource bound with a testable, system-owned enforcement seam** — the "bounded disk" NFR rested on ffmpeg's incidental cleanup with no enforcing path. | software-architect | notes (done) | high | Architect |
| 5 | **Assert resource-bound NFRs in the test suite** — treat each NFR (disk/memory/process count) as a test obligation (assert the bound holds, e.g. session dir cleared on stop), and add negative tests for dependency-death/unbounded-wait. | quality-engineer | notes (done) | high | QA |
| 6 | **No wait-state may be unbounded** — any wait on an external process/resource must poll the dependency's liveness/failure and transition to an explicit error state within a bound (M1 "Preparing forever"). | senior-software-engineer | notes (done) | med | Senior Eng |
| 7 | **Resolve the no-Bash narration gap** — PM, researcher, UX/UI have no Bash/gh tool and can't self-narrate on issues; the Orchestrator narrates for them. Either grant a scoped narration capability or formalize "Orchestrator narrates for no-Bash agents." **RECURRING/unresolved** (noted at team-pulse). | orchestrator (+ affected personas) | process decision; **escalate** | med | Process Eng / Orchestrator |
| 8 | **Reinforce: gate 5 earns its keep** — both projects, the review gate caught real release-blocking bugs tests missed (here H1/M1/M2 + 4 security). Keep BLOCK → fix → re-review discipline; no change needed, recorded as positive signal. | reviewer-critic / security-engineer | — (reinforcement) | — | Process Eng |
| 9 | **Reinforce: our own process fixes paid off** — the post-team-pulse start-project changes (gate-1-before-repo, interview-in-main, dir auto-registration) produced a clean gate 1 and zero directory-access prompts. The self-improvement loop is working; keep verifying applied fixes on the next run. | orchestrator | notes (done) — positive signal | — | Process Eng |

## 4. Actions taken

| Recommendation | Result | Link |
|----------------|--------|------|
| #1 | Notes appended to researcher-analyst; flagged as candidate contract item (route to gated PR) | `.claude/agents/researcher-analyst/notes.md` |
| #2 | Notes appended to senior-software-engineer + security-engineer; flagged as candidate contract / gate-3 build-DoD change (route to gated PR) — **RECURRING** | `.claude/agents/senior-software-engineer/notes.md`, `.claude/agents/security-engineer/notes.md` |
| #3 | Notes appended to orchestrator; flagged as skill fix (start-project/run-project/smart-merge) — supersedes team-pulse rec #4 | `.claude/agents/orchestrator/notes.md` |
| #4 | Notes appended to software-architect | `.claude/agents/software-architect/notes.md` |
| #5 | Notes appended to quality-engineer | `.claude/agents/quality-engineer/notes.md` |
| #6 | Notes appended to senior-software-engineer | `.claude/agents/senior-software-engineer/notes.md` |
| #7 | Escalated as a process decision (capability grant vs explicit convention); notes appended to orchestrator — **RECURRING/unresolved** | `.claude/agents/orchestrator/notes.md` |
| #8 | Recorded as positive signal; no change | this report |
| #9 | Recorded as positive signal; note appended to orchestrator | `.claude/agents/orchestrator/notes.md` |

### Routed follow-ups — ✅ ALL APPLIED 2026-06-04 (human-approved)

The operator approved applying all contract/skill recommendations (the human side of the DESIGN §5
gate). Applied in merge "apply hearthflix post-mortem recommendations":

- ✅ **Contract + gate-3 DoD (rec #2, RECURRING):** secure-by-default build checklist (security
  headers/CSP, dependency CVE audit, non-root container, loopback, no-unbounded-wait, resource
  teardown) added to `senior-software-engineer/persona.md` **and** to `docs/gates.md` gate-3 DoD.
- ✅ **Contract (rec #1):** validate-don't-assert (label `validated` vs `assumed`; NFR-bound claims
  must be demonstrated) added to `researcher-analyst/persona.md`.
- ✅ **Skill (rec #3):** programmatic post-merge close-loop now specified in `start-project`,
  `run-project`, and `smart-pull-request`; PR-keyword reliance dropped.
- ✅ **Process decision (rec #7):** no-Bash narration gap resolved — DESIGN §6 + `run-project` now
  state the Orchestrator posts narration on behalf of agents lacking `gh` (they return the text).

## 5. Metrics (signal)

- **Gates passed first try:** **6/7.** Gates 1, 2, 3, 4, 6 passed first try; gate 5 took one fix
  loop (BLOCK on H1 + M1/M2 + 4 security findings → build → clean re-review). *(Up from 5/7 at
  team-pulse — gate 1 no longer needed a re-round, thanks to applied process fixes.)*
- **Test pass rate / coverage:** 59/59 passing (31 at build → 49 at gate 4 with QA's +18 negative/
  edge tests → 59 after the fix loop); all ACs mapped, AC6/AC11 documented as manual.
- **Review findings:** 3 HIGH/MED blockers from the Reviewer (H1 unbounded disk, M1 "Preparing
  forever", M2 remux stall) + 4 security findings (FIND-01 headers/CSP, FIND-02 CVE deps, FIND-03
  root container, FIND-04 open bind) + lows; all remediated, both reviewers re-verified PASS.
- **Notable rework loops:** 1 (gate 5 BLOCK → build → re-review). No discovery re-round.
- **Escaped defects:** **0** to v0.1.0 — operator's real Arch + Docker acceptance run passed
  **all 13 ACs first try** (byte-identical media checksums proved read-only; pass-through confirmed
  no-ffmpeg; HEVC transcoded; remux verified; LAN/no-auth confirmed).
- **Backlog filed:** 3 non-blocker observations → v1.1 (#21 HEAD/405, #22 live-confirm transcode
  cap, #23 add procps).
- **Recurring lessons:** 2 — secure-by-default-at-first-write (rec #2) and the no-Bash narration gap
  (rec #7). Multi-issue auto-close recurred and **superseded** team-pulse's fix (rec #3).
- **Positive signals:** the applied team-pulse process fixes worked (clean gate 1, zero dir-access
  prompts); gate 5 again caught all release-blocking defects pre-release.
