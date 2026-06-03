# Post-mortem: team-pulse-dashboard

- **Project:** team-pulse-dashboard · **Repo:** https://github.com/CKS-Equium/team-pulse-dashboard
- **Dates:** 2026-06-03 → 2026-06-03 (v0.1.0)
- **Facilitator:** Process Engineer
- **Outcome:** Shipped (v0.1.0) — the team's first dogfood project

> A read-only, auto-refreshing local web dashboard streaming real GitHub activity across watched
> projects, surfacing `needs-human` items so the operator can click straight to the issue and
> unblock the team. Participating roles: orchestrator, product-manager, software-architect,
> ux-designer, ui-designer, project-manager, senior-software-engineer, quality-engineer,
> reviewer-critic, security-engineer, devops-release-engineer, technical-writer. The junior
> engineer was not separately spawned — the senior built directly.

## 1. Self-assessments

### Orchestrator
- **Kept:** Ran the live operator interview in the main thread, then handed a clean transcript to
  the PM; enforced gates and drove the gate-5 fix loop back through build to re-review.
- **Dropped / would change:** start-project created the repo and registry *before* gate-1 PRD
  approval (orphan-repo risk), and didn't auto-register the new project dir, causing
  directory-access prompts for sub-agents.
- **Lesson → notes.md:** Live interview stays in the main thread; sequence gate-1 before repo
  creation; auto-register new project dirs in settings; use one closing keyword per issue.
  *(confidence: high)*

### Product Manager
- **Kept:** Drafted a testable PRD with all ACs mapped, from the Orchestrator's captured Q&A.
- **Dropped / would change:** First framing was a static status snapshot; gate 1 took 2 rounds
  before the operator reframed it to a live activity log + "Needs-you" panel.
- **Lesson → notes.md:** Author PRDs from the captured interview transcript; probe "what action
  does this drive?" in round 1 for ops/monitoring tools. *(confidence: med)*

### Software Architect
- **Kept:** Gate 2 approved first try — Node backend + static SPA, token backend-only, 20s polling
  with ETag + back-off, loopback bind, label-only `needs-human` detection.
- **Dropped / would change:** Interface examples used `owner/name` while the registry stores `repo`
  as a URL; the "phase: registry authoritative, conflict flagged" rule under-specified how
  milestone is derived.
- **Lesson → notes.md:** Pin shared-identifier shape and normalization point in the contract;
  specify the filter and resolution rule for derived state. *(confidence: med)*

### Senior Software Engineer
- **Kept:** Delivered all 10 tickets as PR #15, building locally; turned the gate-5 findings around
  quickly to a clean re-review.
- **Dropped / would change:** Shipped a `javascript:` XSS sink (no scheme check), missing
  CSP/headers, false phase-conflict (derived from a closed/arbitrary issue), hardcoded active-role
  ("orchestrator"), and a silently-empty in-progress event class — all caught at review, not first
  write.
- **Lesson → notes.md:** Scheme-allowlist external hrefs + CSP + loopback on first write; derive
  state from the right subset (open issues, role label); never render an event class silently
  empty. *(confidence: high)*

### Quality Engineer
- **Kept:** Grew the suite 46→55 with every AC mapped; gate 4 passed cleanly.
- **Dropped / would change:** The multi-milestone test was too weak to catch the false
  phase-conflict (the Reviewer caught it instead); live contract tests depend on an externally-run
  server and are flaky in CI.
- **Lesson → notes.md:** Test negative/noisy derived-state cases and assert the resolved value;
  contract tests must own their server lifecycle. *(confidence: high)*

### Security Engineer
- **Kept:** BLOCKed gate 5 on a HIGH `javascript:` XSS sink and flagged missing CSP/headers and a
  non-loopback bind; re-reviewed to PASS after fixes.
- **Dropped / would change:** Nothing structural — the gate worked. Codify the data-driven-UI
  checklist so the sink is caught at design/build rather than review.
- **Lesson → notes.md:** Scheme-allowlist any external-data URL + CSP + loopback; keep blocking on
  HIGH. *(confidence: high)*

### DevOps / Release Engineer
- **Kept:** CI workflow in place; gate-6 release tagged v0.1.0 cleanly; rollback path known.
- **Dropped / would change:** `Closes #5, #6, …` only auto-closed #5 (rest closed manually); CI
  inherited contract tests that assume an externally-running server (flaky).
- **Lesson → notes.md:** One closing keyword per issue (or script it); CI test jobs must own their
  runtime dependencies. *(confidence: high)*

### Reviewer / Critic
- **Kept:** Caught defects that tests missed — the false phase-conflict badge and the wrong
  active-role — making gate 5 the net that worked. No durable notes lesson this run (performed to
  contract); feedback routed as 360 input to QA and Engineering.

### Project Manager
- **Kept:** Clean 10-ticket plan with dependencies; build tracked to delivery. No durable
  notes-level lesson this run.

### UX Designer / UI Designer
- **Kept:** Flows and visual specs cleared gate 2 first try and needed no rework. No durable
  notes-level lesson this run.

### Technical Writer
- **Kept:** README/changelog/run instructions (`npm install && npm start`) shipped with the
  release. No durable notes-level lesson this run.

## 2. 360 reviews (chain neighbors)

### Orchestrator → Product Manager
- **Worked well:** PM consumed the interview transcript and produced a testable PRD without needing
  a live channel.
- **Friction / suggestion:** First framing was static and cost a second gate-1 round — probe the
  action the tool drives earlier.

### Product Manager → Orchestrator
- **Worked well:** Running the live interview in the main thread and handing over a transcript was
  the right division of labour.
- **Friction / suggestion:** Repo was created before PRD approval; defer it until gate 1 passes.

### Orchestrator → Software Architect
- **Worked well:** Gate 2 cleared first try; the design was buildable.
- **Friction / suggestion:** Tighten under-specified derived-state and identifier-shape details
  that surfaced as build/review defects.

### Software Architect → Senior Software Engineer
- **Worked well:** Faithful implementation of the architecture; fast fix turnaround at gate 5.
- **Friction / suggestion:** Several issues (XSS sink, false conflict, hardcoded role) were
  first-write defects, not architecture gaps — apply secure-by-default and correct derived-state
  resolution proactively.

### Senior Software Engineer → Software Architect
- **Worked well:** Clear stack and NFR decisions.
- **Friction / suggestion:** Contract left identifier shape (`owner/name` vs URL) and the
  milestone-derivation rule ambiguous; pin both.

### Quality Engineer → Senior Software Engineer
- **Worked well:** Code was testable and ACs were addressable.
- **Friction / suggestion:** Derived-state logic needed negative-case attention; coordinate on a
  self-contained test harness.

### Reviewer/Critic → Quality Engineer
- **Worked well:** Strong AC coverage (55/55).
- **Friction / suggestion:** The multi-milestone test was too weak to catch the false conflict —
  assert resolved values on noisy inputs so the Reviewer isn't the only net.

### Security Engineer → Senior Software Engineer
- **Worked well:** Remediated the HIGH and MEDs correctly and fast.
- **Friction / suggestion:** Scheme-check external URLs and add CSP/loopback on first write — these
  are predictable for any data-driven web UI.

### Reviewer/Critic + Security Engineer → Orchestrator
- **Worked well:** Gate 5 was respected — the fix loop ran back through build to a clean re-review
  before release.
- **Friction / suggestion:** None on gate discipline; the gate is doing its job.

### DevOps → Quality Engineer
- **Worked well:** Suite green at release time.
- **Friction / suggestion:** Contract tests depend on an external server — refactor to be
  self-contained before treating CI as an authoritative release gate.

## 3. Improvement recommendations

| # | Recommendation | Target role | Level (notes / contract / skill) | Severity | Owner |
|---|----------------|-------------|----------------------------------|----------|-------|
| 1 | start-project must run the live discovery interview in the Orchestrator's main thread (a PM subagent has no interactive channel); PM drafts the PRD from the captured transcript | orchestrator / product-manager | skill (start-project Phase 2) | high | Orchestrator |
| 2 | Reorder start-project so gate-1 PRD approval precedes `gh repo create` (no orphan repos) | orchestrator | skill (start-project) | high | Orchestrator |
| 3 | start-project should auto-register each new project dir into `settings.local.json` so spawned agents avoid directory-access prompts | orchestrator | skill (start-project) + settings | med | Orchestrator |
| 4 | Use one closing keyword per issue (`Closes #5, Closes #6, …`) or script closes-on-merge | devops-release-engineer | notes (done) + skill (smart-pull-request/smart-merge) | med | DevOps |
| 5 | Refactor live contract tests to own their server lifecycle (start/stop a fixture, ephemeral port); never depend on an externally-running process | quality-engineer | notes (done) + project follow-up | high | QA |
| 6 | Test negative/noisy cases for derived state (closed issues, no open milestone) and assert the resolved value, not just branching | quality-engineer | notes (done) | high | QA |
| 7 | Default to a scheme allowlist (http/https) for any external-data href, add CSP/security headers, bind local tools to loopback — at first write | senior-software-engineer | notes (done) | high | Senior Eng |
| 8 | Add a data-driven-web-UI security checklist (URL scheme allowlist, CSP, loopback bind) to the standard security review | security-engineer | notes (done); candidate contract item | med | Security |
| 9 | Probe "what action does this tool drive?" in round 1 of discovery for ops/monitoring tools (avoid static-snapshot framings) | product-manager | notes (done) | med | PM |
| 10 | Pin shared-identifier shape and the normalization boundary in interface contracts; specify the filter + conflict-resolution rule for derived state | software-architect | notes (done) | med | Architect |

## 4. Actions taken

| Recommendation | Result | Link |
|----------------|--------|------|
| #1 | Routed as proposed skill change (start-project Phase 2); notes appended to orchestrator + product-manager | `.claude/agents/orchestrator/notes.md`, `.claude/agents/product-manager/notes.md` |
| #2 | Routed as proposed skill change; notes appended to orchestrator | `.claude/agents/orchestrator/notes.md` |
| #3 | Routed as proposed skill + settings change; notes appended to orchestrator | `.claude/agents/orchestrator/notes.md` |
| #4 | Notes appended to devops-release-engineer; skill change proposed | `.claude/agents/devops-release-engineer/notes.md` |
| #5 | Notes appended to quality-engineer + devops; flagged as project follow-up | `.claude/agents/quality-engineer/notes.md`, `.claude/agents/devops-release-engineer/notes.md` |
| #6 | Notes appended to quality-engineer | `.claude/agents/quality-engineer/notes.md` |
| #7 | Notes appended to senior-software-engineer | `.claude/agents/senior-software-engineer/notes.md` |
| #8 | Notes appended to security-engineer; flagged as candidate contract/checklist item | `.claude/agents/security-engineer/notes.md` |
| #9 | Notes appended to product-manager | `.claude/agents/product-manager/notes.md` |
| #10 | Notes appended to software-architect | `.claude/agents/software-architect/notes.md` |

### Routed follow-ups requiring the gated path — ✅ APPLIED 2026-06-03 (human-approved)

The operator approved applying all recommendations, which is the human side of the contract gate
(DESIGN §5). Applied:

- ✅ **Skill: start-project** (recs #1, #2, #3) — rewritten: live interview in the main thread;
  gate-1-before-repo ordering; auto-registers each new project dir in `settings.local.json`; plus
  the closing-keyword convention. *(control-plane merge "apply post-mortem recommendations")*
- ✅ **Skill: smart-pull-request** (rec #4) — one-closing-keyword-per-issue rule added.
- ✅ **Contract: security-engineer `persona.md`** (rec #8) — data-driven-web-UI security checklist
  promoted into the contract (human approved the gate, overriding the "wait for a 2nd project"
  caution).
- ✅ **Project: team-pulse-dashboard** (rec #5) — contract tests refactored to own their server
  lifecycle; CI simplified; 55/55 pass with no external server. *(merged to project `main`)*
- ✅ **gates.md** — `needs-human` async-escalation mechanism documented.

**Still open (forward feature, not a fix):** the `needs-human` **scheduled-resume cron routine**
(via the `schedule` skill) — convention adopted, auto-resume not yet built; escalations checked
manually until then.

## 5. Metrics (signal)

- **Gates passed first try:** 5/7. Gate 1 took 2 rounds (static → live reframe); gate 5 took a fix
  loop (security BLOCK on HIGH + Reviewer findings → build → clean re-review). Gates 2, 3, 4, 6
  passed first try.
- **Test pass rate / coverage:** 55/55 passing (suite grown 46→55); all acceptance criteria mapped.
- **Review findings:** 1 HIGH (`javascript:` XSS sink) + 2 MED + several low security/review
  findings; all remediated, security re-review PASS.
- **Notable rework loops:** 1 (gate 5 → build → re-review). One discovery re-round at gate 1.
- **Escaped defects:** 0 to v0.1.0 — gate 5 caught all known defects pre-release.
- **Process gaps found:** 4 tooling/skill issues (interview-in-main-thread, repo-before-gate-1,
  dir-registration, multi-issue auto-close) + 1 test-harness coupling.
