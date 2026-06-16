# Post-mortem: new-cadair (PM-team A/B build of Cadair Phase A)

- **Project:** new-cadair · **Repo:** https://github.com/CKS-Equium/new-cadair
- **Dates:** 2026-06-16 (full Phase A A.1→A.6 in one autonomous run; experiment concluded)
- **Facilitator:** Process Engineer
- **Outcome:** Complete — feature **and run** parity with the OG-cadair reference; **0 escaped defects**

> **What this is (and isn't).** This is the structured **team retrospective on *how the team
> executed*** the new-cadair build. It is deliberately distinct from the
> **[A/B experiment report](new-cadair-ab.md)**, which is the outcome/comparison artifact (the
> head-to-head, the 5 defects, the 3 non-obvious lessons). That report answers *"did the improved
> process pay out?"* — **yes.** This report answers *"how did the team work, where did it strain,
> and what should graduate from notes → contract?"* It **builds on** the A/B report; it does not
> repeat the head-to-head table.
>
> **The experiment's product is the PM team, not Cadair.** new-cadair was the PM team's independent
> build of **Cadair Phase A only**, from the *same controlled specs* as OG-cadair (`D:/github/cadair`),
> with OG's implementation + phase-design **deliberately withheld** so the team designed Phase A
> itself. The headline process bets — the per-gate **evidence record**, **requirement→test→source
> traceability**, and the **failure-mode review lens** — were already banked (operator-approved
> 2026-06-16) *before* this build, so new-cadair was produced by the improved team and is the field
> trial of those three contract additions.
>
> **Participating roles:** **orchestrator** (drove the run; branch+PR per sub-phase; independent
> per-gate DoD verification; the A.6 recovery), **software-architect** (designed Phase A from the FS +
> architecture, fairness-withheld from OG's design), **senior-software-engineer** (built every
> sub-phase A.1–A.6, ~231 Phase-A tests), **quality-engineer lens** (evidence records + the
> consolidated RTM + the no-mocks system-validation gate; coverage authored by the engineer),
> **reviewer-critic** (the failure-mode pass — the 5 pre-merge defects). product-manager,
> ux/ui-designer, devops-release-engineer, and security-engineer were **not separately engaged**:
> there was no live PRD/UX phase (the FS + design were the controlled spec), no hosted deploy
> (loopback demo), and the security baseline was applied inline by the engineer at authoring time.

> **Reading note:** lessons marked **RECURRING** also appeared in a prior post-mortem (colonygame /
> hearthflix / team-pulse) — the strongest candidates for contract promotion.

## 1. Self-assessments

### Orchestrator
- **Kept:** Drove the whole run under the sub-phase cadence — **branch + PR per sub-phase, self-merge
  on green DoD**, with **independent per-gate DoD verification** (ran `ci.sh` itself; read each
  evidence record for honesty rather than trusting the agent's claim). **0 escaped defects.** The
  standout: on **A.6 the building agent died twice on transient API errors (500, then 529)** with no
  final report, and the run still shipped — the orchestrator re-verified the half-built A.6 from its
  **durable artifacts alone** (ran `ci.sh` → 7/7 green, 158 tests, full suite 5× → 0 flakes; read the
  evidence doc → confirmed the 12 deferrals were *named/routed*, not hidden) and merged it as PR #6
  **without resurrecting the dead agent's context.**
- **Dropped / would change:** The recovery *worked*, but it was only possible because the sub-phase
  had already left a one-command verifier + an honest evidence record — that property was a happy
  consequence of the banked process, **not an explicit, named requirement** the orchestrator could
  rely on. There is no written contract that says "every sub-phase must be independently re-verifiable
  from its artifacts after agent death." The recovery also leaned on the orchestrator personally
  knowing to run `ci.sh` and read the evidence doc; a fresh agent or the human would have had to
  rediscover that path.
- **Lesson → notes.md:** Make "**one-command verifier + self-describing evidence record per
  sub-phase**" an explicit **interruption-tolerance / handoff** requirement, not just a CI nicety —
  any actor (fresh agent, orchestrator, human) must be able to independently verify and integrate a
  sub-phase from its durable artifacts. **Candidate gate-DoD addition.** *(confidence: high)*

### Software Architect
- **Kept:** Designed Phase A from the FS + architecture with **OG's phase-design withheld** — and the
  team still reached feature+run parity, which is the cleanest possible evidence that the spec set was
  sufficient and the design was sound. Notable **fork discipline**: pyo3 0.23→0.25.1 (Python 3.14
  support), pymodbus 3.13 API surface, pyzmq-27-no-tornado — each was a *minor/patch* move *within the
  decided stack*, recorded with rationale in the evidence "Forks / decisions" sections, never a
  silent stack change.
- **Dropped / would change:** Two design-time blind spots surfaced only at review/run, not at design:
  (a) the **HOLD-on-UNCERTAIN vs HOLD-on-BAD** ambiguity (A.1 M2) was an internal FS inconsistency
  the design did not resolve up front — it shipped, was caught at review, and resolved spec-faithfully;
  (b) the **metric-span** decision (engine-only `scan_time_ms`) was never made an explicit design
  contract, so the live telemetry measured a different span than OG's and looked 1000× faster while
  hiding adapter+publish latency (A/B report §4.2).
- **Lesson → notes.md:** Resolve **internal spec inconsistencies at design time** (a single
  authoritative rule for each behaviour) so they aren't discovered as review deviations; and when a
  metric is a **budget/gate**, the *design* should declare the **span it covers** (prefer end-to-end).
  *(confidence: med)*

### Senior Software Engineer *(the workhorse — all six sub-phases)*
- **Kept:** Built **every** sub-phase end-to-end — Rust engine (zero-alloc, ~0.94 µs scan), Modbus
  adapter (live in-process server, no mocks), ZeroMQ bus (real sockets + durable SQLite queue),
  UFDF simulator + **real closed-loop control over a real Modbus socket**, the FastAPI/React HMI, and
  the one-command CI — growing to **158 pytest + 73 cargo = 231 Phase-A tests**. **Honest deferral
  discipline** throughout: 12 of 104 FS-IDs **named-deferred** with routing (e.g. RBAC→C, alarm
  routing→E), and FS-PROT-011 **re-scoped to PARTIAL** (alarm→E, audit→C) rather than overclaimed.
  Secure-by-default applied at authoring time (CSP/headers, `npm audit` 0, loopback bind, bounded
  tasks).
- **Dropped / would change:** **RECURRING — review-fix churn that a sharper authoring pass would have
  pre-empted.** Three of the five defects the reviewer caught were on the *first* sub-phase (A.1: B1
  interlock-trip event flooding, M1 host-abort on oversized config, M2 HOLD-on-UNCERTAIN), and A.2
  needed a BLOCKER + MAJOR round, A.3 a MAJOR. Every one is a degenerate-input / latched-state /
  dependency-death / lock-scope case — i.e. **exactly the failure-mode lens the reviewer applies at
  review.** The engineer green-built and submitted; the lens was applied *to* the submission rather
  than *during* it. The same lens, run as a self-check before requesting review, would have caught
  most of these at authoring time and saved the fix rounds.
- **Lesson → notes.md:** Run the **failure-mode lens on your own diff as part of self-review** for
  control/safety/data-integrity/resource-bound code, *before* requesting review — the review lens
  should confirm, not discover, the degenerate-case handling. *(confidence: high)* — **candidate
  contract.**

### Quality Engineer *(lens — RTM + evidence records + the no-mocks validation gate)*
- **Kept:** The **evidence record per sub-phase** + the **consolidated RTM** (104 FS-IDs, 92 traced /
  12 named-deferred, CI-completeness-gated by `check_rtm.py`) were exactly the auditable trail the
  banked process intended, and they **doubled as the resilience asset** that let the orchestrator
  recover A.6. The **system-validation gate boots the real stack with no mocks** and asserts a live WS
  frame round-trips. The flaky bus test was **root-caused (`drain_at_least`), not slept-around**, and
  proven deterministic 5×.
- **Dropped / would change:** Two coverage blind spots that *green numbers hid* (A/B report §4.2,
  §4.3): (a) **metric-span** — the engine-only `scan_time_ms` is "green" but measures the wrong span
  vs the 100 ms loop budget; the RTM/evidence never declared *what span the number covers*; (b) the
  **operator-path gap** — the validation gate proves the data path (sim→engine→bus→WS) but the recipe
  **auto-runs on boot**, so the **operator-initiated start path is never exercised** by any gate. Both
  passed green; neither was a real proof of the thing the requirement is about.
- **Lesson → notes.md:** (i) Add a **metric-span honesty** sibling to the failure-mode lens — for any
  metric used as a budget/gate, the evidence/RTM must declare the **span it covers** and prefer
  **end-to-end**; (ii) the system-validation gate must exercise the **primary operator-initiated path**
  (or explicitly register the gap), not only an auto-driven one. **Both candidate contracts.**
  *(confidence: high)*

### Reviewer / Critic *(earned its keep — the headline signal)*
- **Kept:** The **failure-mode pass caught 5 real, shippable defects the green suites missed**,
  pre-merge, on the Tier-1 / data-integrity sub-phases — **the strongest single signal of the
  experiment** and the field-validation of the lens contract banked the same day: interlock-trip event
  flooding (A.1 B1), host-process abort on oversized config (A.1 M1), HOLD-on-UNCERTAIN deviation
  (A.1 M2), unguarded event-sink escape / silent reconnect-task death (A.2 BLOCKER), and `publish()`
  lock-scope thread-safety + persist-order divergence (A.3 MAJOR). Each turnaround landed with a
  **teeth-proven** regression test (red-without-fix). Also surfaced **metric-span** as a new
  *misleading-evidence* defect class while reviewing the two builds' telemetry.
- **Dropped / would change:** The lens is currently a **review-only** gate. Because the engineer did
  not self-apply it, the review *discovered* the defects rather than confirming their handling — fine
  for catching them, but it converts every degenerate-case miss into a fix round. The reviewer would
  prefer the lens be a **shared standing checklist applied at both authoring and review**.
- **Lesson → notes.md:** Keep interrogating *what a green metric measures* (metric-span honesty), and
  push the failure-mode lens **upstream** to authoring so review confirms rather than discovers.
  *(confidence: high)*

### Roles NOT separately engaged (and why)
- **Product Manager / UX-UI Designer:** the **FS + architecture + HMI design were the controlled
  spec** (A/B input), so there was no live discovery/PRD or UX-authoring phase. The PM did capture the
  operator-path acceptance lesson in its notes (the AC angle of §4.3). HMI work followed the supplied
  `hmi-design-spec.md`.
- **DevOps / Release Engineer:** no hosted deploy — `run-demo.sh` + a one-command `ci.sh` on loopback;
  a GitHub-Actions wrapper around `ci.sh` is a named follow-up.
- **Security Engineer:** secure-by-default was applied **inline at authoring** (CSP + security headers,
  authed-before-upgrade WebSocket, HttpOnly/SameSite session cookie, constant-time compare, `npm audit`
  0, loopback bind, bounded tasks); no separate adversarial security review was run for a localhost
  demo with no external attack surface. HTTPS/TLS + real RBAC/e-sig are named Phase C/H deferrals.

## 2. 360 reviews (chain neighbours)

### Orchestrator → Senior Software Engineer
- **Worked well:** Every sub-phase arrived as a self-contained, green, branch+PR unit with an honest
  evidence record and named deferrals — that made independent DoD verification *and* the A.6 recovery
  possible.
- **Friction / suggestion:** The A.1 B1/M1/M2 + A.2 BLOCKER/MAJOR + A.3 MAJOR fix rounds were all
  degenerate-case handling — run the failure-mode lens on your **own** diff before requesting review
  so the reviewer confirms rather than discovers.

### Senior Software Engineer → Orchestrator
- **Worked well:** Independent per-gate verification (running `ci.sh`, reading the evidence) kept the
  bar honest without a human watching; the A.6 recover-from-artifacts move (don't resurrect a dead
  agent's context) is the right pattern for long autonomous runs.
- **Friction / suggestion:** The recovery worked by luck of the banked process — make
  "independently re-verifiable from artifacts" an *explicit* sub-phase DoD so it's relied on, not
  rediscovered.

### Reviewer / Critic → Senior Software Engineer
- **Worked well:** Fix turnarounds were fast, complete, and every regression test had **proven teeth**;
  the FS-PROT-011 honest re-scope (PARTIAL, not overclaimed) is exactly the integrity wanted.
- **Friction / suggestion:** The five defects were all in the lens's named families — self-apply the
  lens at authoring so they're handled on first submission.

### Senior Software Engineer → Reviewer / Critic
- **Worked well:** Caught five genuinely shippable correctness/safety bugs a green build would have
  shipped, and the new metric-span catch — the adversarial lens is why 0 defects escaped.
- **Friction / suggestion:** None on the review itself; the right fix is upstream (authoring-time
  self-check), not in the review.

### Quality Engineer ↔ Reviewer / Critic (on metric-span)
- **Worked well:** The reviewer's metric-span catch and the QA RTM/evidence discipline are two halves
  of the same principle — green is *evidence*, not proof.
- **Friction / suggestion:** Bake the span-declaration into the evidence/RTM template so a perf number
  can't be recorded without stating what it times.

### Orchestrator + Quality Engineer → the system-validation gate (process self-review)
- **Worked well:** The no-mocks gate booting the real stack is a strong proof of the data path and a
  required-every-CI artifact.
- **Friction / suggestion:** It proved the *machine* path; because the recipe auto-runs, the
  **operator's first action** went unverified. The gate must drive the operator-initiated path or
  register the gap (RECURRING — echoes the colonygame pre-playtest "verify the path the human takes").

## 3. Improvement recommendations

| # | Recommendation | Target role | Level | Severity | Owner |
|---|----------------|-------------|-------|----------|-------|
| 1 | **Failure-mode lens at AUTHORING time, not only at review** — the engineer self-runs the lens (BAD/NaN/Inf/div0/boundary/latched/dependency-death/unbounded-wait) on its own diff for control/safety/data-integrity/resource-bound code before requesting review, so review *confirms* rather than *discovers*. All 5 pre-merge defects were in these named families. | senior-software-engineer | notes (done); **candidate contract** | high | Senior Eng |
| 2 | **Metric-span honesty** — for any metric used as a budget/gate (scan time, latency, throughput), the evidence/RTM must declare the **span it covers** and prefer the **end-to-end** span; an engine/bench-only figure presented as the cycle figure is a misleading-evidence defect. Add as a sibling to the failure-mode lens. | reviewer-critic + quality-engineer (+ gate 4 / evidence template) | notes (done); **candidate contract** | high | Reviewer / QA |
| 3 | **System-validation gate must exercise the primary operator-initiated path** (or explicitly register the gap) — an auto-driven demo proves the machine path and leaves the human's actual first action unverified. RECURRING (colonygame). | quality-engineer + product-manager (+ gate 5 / playtest-gate text) | notes (done); **candidate contract** | high | QA / PM |
| 4 | **One-command verifier + self-describing evidence record = a named interruption-tolerance / handoff requirement per sub-phase**, not just a CI nicety — any actor must be able to independently re-verify and integrate a sub-phase from its durable artifacts after agent death. Proven by the A.6 double-agent-death recovery. | process-engineer + orchestrator (+ gate-DoD / evidence-record §) | notes (done); **candidate gate-DoD** | med | Process Eng / Orchestrator |
| 5 | **Resolve internal spec inconsistencies at design time** (one authoritative rule per behaviour) so they aren't discovered as review deviations — the HOLD-on-UNCERTAIN vs HOLD-on-BAD ambiguity (A.1 M2) shipped because the FS was internally inconsistent and the design didn't reconcile it up front. | software-architect | notes (suggested) | med | Architect |
| 6 | **Reinforce — failure-mode lens earns its keep (FIELD-PROVEN):** 5 real shippable defects caught pre-merge that green suites missed, on the first build by the improved team. The banked lens contract is validated; keep it standing for Tier-1 / data-integrity / compliance code. | reviewer-critic | positive signal | — | Process Eng |
| 7 | **Reinforce — honest deferral / scoping discipline:** 12/104 FS-IDs named-deferred with routing and FS-PROT-011 re-scoped to PARTIAL rather than overclaimed; the evidence record's known-follow-ups register is being used as intended. Keep. | senior-software-engineer / quality-engineer | positive signal | — | Process Eng |

## 4. Actions taken

| Recommendation | Result | Link |
|----------------|--------|------|
| #1 | Lesson appended to senior-software-engineer notes (engineer self-review angle); flagged **candidate contract** → operator approval. Currently the lens lives in the reviewer-critic + gates.md (review-only). | `.claude/agents/senior-software-engineer/notes.md` (to add); this report §5 |
| #2 | Notes appended to quality-engineer and reviewer-critic (metric-span honesty); flagged **candidate contract** → operator approval. | `.claude/agents/quality-engineer/notes.md`, `.claude/agents/reviewer-critic/notes.md` |
| #3 | Notes appended to quality-engineer and product-manager (operator-path); flagged **candidate contract** → operator approval. | `.claude/agents/quality-engineer/notes.md`, `.claude/agents/product-manager/notes.md` |
| #4 | Notes appended to process-engineer and orchestrator (evidence-as-resilience); flagged **candidate gate-DoD** → operator approval. | `.claude/agents/process-engineer/notes.md`, `.claude/agents/orchestrator/notes.md` |
| #5 | Recorded; suggested note for software-architect (no architect notes entry exists for this run). | this report |
| #6 / #7 | Recorded as positive signals; the lens contract (banked 2026-06-16) is field-validated; no change needed. | `.claude/agents/reviewer-critic/persona.md`, `.claude/agents/quality-engineer/persona.md` |

### Routed follow-ups — ⏳ AWAITING OPERATOR APPROVAL

Per DESIGN §5 + gates.md, contract / gate-DoD changes need the human side of the gate. **No
`persona.md` or `gates.md` was edited by this post-mortem.** Recommendations #1–#4 are flagged as
contract / gate-DoD candidates; proposed exact wording is in §5 below so the contract-promotion step
(a reviewed PR + operator approval) can use it verbatim. Recommendations #6/#7 are reinforcing
signals — no change.

## 5. Contract-promotion recommendation — which lessons graduate, with exact wording

Three lessons are proven and high-confidence enough to graduate **notes → contract**; a fourth is a
gate-DoD addition. The metric-span and operator-path lessons appear in *two* agents' notes
independently (QA + reviewer; QA + PM), and the operator-path lesson is **RECURRING** from colonygame —
both strong promotion signals. The failure-mode-at-authoring item is a new contract item for the
engineer, derived directly from the fact that all five caught defects were in the lens's families.

> **Process Engineer's explicit recommendation:** graduate **#1, #2, and #3** to contracts and **#4**
> to the gate-DoD. #5 stays a note (single occurrence). #6/#7 are reinforcing — the lens contract is
> already in place and is now field-validated; no edit.

### [contract] #1 — Failure-mode lens at authoring time (senior-software-engineer)
*Rationale:* All 5 pre-merge defects (A.1 ×3, A.2, A.3) were degenerate-input / latched / dependency-death
/ lock-scope cases — the reviewer's lens applied *after* submission. Self-applying it before review
converts most fix rounds into first-pass-clean. The reviewer-critic already carries the lens (review
side); this is the **authoring** counterpart.

**Proposed addition to `.claude/agents/senior-software-engineer/persona.md`** (new subsection):
> ## Failure-mode self-review (authoring side of the review lens)
> Before requesting review on **control/safety-relevant, data-integrity, or resource-bound** code,
> run the **failure-mode lens on your own diff** — the same checklist the Reviewer/Critic applies at
> gate 5: BAD / invalid input · NaN / Inf · divide-by-zero / degenerate params · boundary values ·
> latched / stuck states · dependency death · unbounded waits. Add a named test for each applicable
> case. The review lens should **confirm** your degenerate-case handling, not **discover** it. *(Added
> from the new-cadair A/B post-mortem, 2026-06-16 — operator-approved.)*

**Proposed gates.md change** — extend the "Failure-mode pass (gate 5)" section with a sentence:
> The same lens is run by the **Senior Engineer as a self-review at gate 3** (authoring side), so the
> gate-5 reviewer pass *confirms* rather than *discovers* the degenerate-case handling.

### [contract] #2 — Metric-span honesty (reviewer-critic + quality-engineer)
*Rationale:* Independently surfaced in both the reviewer and QA notes. A green metric measuring the
wrong span (engine-only `scan_time_ms` vs the 100 ms loop budget) is a misleading-evidence defect of
the same family the failure-mode lens catches.

**Proposed addition to `.claude/agents/reviewer-critic/persona.md`** (append to the Failure-mode lens
section, or a sibling line):
> **Metric-span honesty (sibling lens):** for any number presented as meeting a budget/gate
> (scan time, latency, throughput, coverage), interrogate **what span it covers** — an
> engine/bench-only figure presented as the end-to-end/cycle figure is a misleading-evidence defect.
> Prefer the **end-to-end** span; require the span to be stated alongside the number.

**Proposed addition to `.claude/agents/quality-engineer/persona.md`** (add to "Validation-grade
artifacts"):
> - **Metric-span declaration (budget/gate metrics):** when a metric gates a requirement (scan time,
>   latency, throughput), the evidence/RTM entry must declare the **span it covers** and prefer the
>   **end-to-end** span; a bench/engine-only figure must not be recorded as the cycle figure.

### [contract] #3 — Operator-initiated path in the validation gate (quality-engineer + product-manager)
*Rationale:* RECURRING (colonygame "verify the path the human takes"). The no-mocks gate proved
sim→engine→bus→WS but the auto-running recipe meant the operator's "Start Batch" first action was
never exercised.

**Proposed addition to `.claude/agents/quality-engineer/persona.md`:**
> - **Operator-initiated path (system-validation / acceptance):** if the product has a **primary
>   operator entry action** (start batch, submit, confirm), the validation gate must drive **that**
>   action and assert the system responds — not only an auto-driven path — or **explicitly register
>   the gap** in the known-follow-ups register.

**Proposed addition to `.claude/agents/product-manager/persona.md`** (acceptance-criteria authoring):
> Write acceptance criteria around the **operator's primary entry action**, not only the resulting
> data flow — an auto-driven demo otherwise proves the machine path and leaves the human's first step
> unverified.

**Proposed gates.md change** — extend gate 5 / the playtest-gate text:
> Where the product has a **primary operator-initiated entry action**, the system-validation /
> acceptance step must exercise *that* path (or register the gap), not only an auto-driven one.

### [gate-DoD] #4 — Evidence-record + one-command verifier = interruption-tolerance artifact (process-engineer/orchestrator)
*Rationale:* Single occurrence so far, but high-leverage — it is *why the build survived two agent
deaths on A.6.* Stronger as a **gate-DoD framing** of the already-contractual evidence record than as
a new persona rule.

**Proposed addition to `docs/gates.md`** — extend the "Gate evidence record (every gate)" section:
> The evidence record + a **one-command verifier** (`ci.sh` or equivalent) together serve as an
> **interruption-tolerance / handoff** artifact: any actor (a fresh agent, the Orchestrator, the
> human) must be able to **independently re-verify and integrate** a sub-phase from its durable
> artifacts alone — without resurrecting the building agent's context. This makes long autonomous runs
> robust to agent/orchestrator death, not merely auditable.

## 6. Metrics (signal)

- **Sub-phases / PRs:** 6 (A.1–A.6), **one PR each (#1–#6)**, branch+PR, all self-merged on green DoD;
  0 human stops (autonomous run).
- **Tests:** **231 for Phase A** — 158 pytest + 73 cargo; deterministic **5×** (0 flakes); `ci.sh`
  **7/7** stages green from clean.
- **System-validation gate:** boots the **real stack, no mocks**; live WS `scan_complete` frame
  round-trips from the real Modbus simulator; clean idempotent shutdown.
- **Traceability:** consolidated RTM **104 FS-IDs — 92 traced / 12 named-deferred** (CI completeness
  gate, 0 holes); FS-PROT-011 honestly re-scoped to PARTIAL.
- **Defects caught pre-merge vs escaped:** **5 caught pre-merge by the failure-mode lens / 0 escaped**
  (2 in A.1, 1 in A.2, 1 in A.3 — plus M3 doc-accuracy and the metric-span misleading-evidence catch).
- **Review-fix rounds:** **3** sub-phases needed a fix round before merge — A.1 (B1 + M1 + M2 + M3),
  A.2 (BLOCKER-1 + MAJOR-1 + MINOR-1/2/3), A.3 (M1 + m1/m2/m3/m4 + n1); A.4/A.5/A.6 clean on first
  review. All fixes landed with **proven-teeth** regression tests (red-without-fix demonstrated).
- **Forks handled cleanly (no silent stack change):** pyo3 0.23→0.25.1 (Python 3.14), pymodbus 3.13
  API, pyzmq-27-no-tornado, SQLite-WAL compliance queue — each recorded with rationale.
- **Resilience event:** the A.6 building agent **died twice on transient API errors (500, 529)**; the
  sub-phase still shipped because the orchestrator re-verified from durable artifacts (`ci.sh` +
  evidence record) and integrated without resurrecting the agent.
- **Run-parity result (the A/B headline, see [the A/B report](new-cadair-ab.md)):** both stacks boot
  the full stack, serve the HMI, and run a **live closed loop** — OG ~9.2 Hz, new-cadair ~10 Hz; B
  reached **feature + run parity** with the reference from the same specs, OG's design withheld.
- **Recurring lessons:** 1 RECURRING — **verify the operator-initiated path, not just the auto-driven
  one** (also colonygame). The **failure-mode lens earns its keep** signal recurs as a positive (now
  field-proven on the first improved-team build).
- **Remaining open items (named-deferred, not gaps):** RBAC + e-signature (Phase C), HTTPS/TLS
  (Phase H), full ISA-88 recipe (Phase B), GitHub-Actions CI wrapper + coverage % gate (DevOps/H),
  Playwright E2E click-through (QA/B), the **operator-initiated start path** (this report — register
  or test).
