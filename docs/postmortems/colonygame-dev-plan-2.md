# Phase retrospective: colonygame ("Hiraeth") — DEV_PLAN_2 (player-first QoL)

- **Project:** colonygame (Hiraeth) · **Repo:** https://github.com/CKS-Equium/ColonyGame
- **Phase:** DEV_PLAN_2 — player-first QoL (turn the proven T0→T5 sim into a *playable game*)
- **Dates:** 2026-06-08 (single-day QoL run, P1a → P2)
- **Facilitator:** Process Engineer
- **Outcome:** **Phase PARKED in-flight** — P1a → "build & grow flow" merged & playtest-passed; **P2 built,
  machine-verified (234 tests), PR #17 OPEN awaiting playtest (#18) when parked**

> A **phase retrospective**, not a project-closure post-mortem — the original T0→T5 sim build had its own
> gate-7 post-mortem (`docs/postmortems/colonygame.md`). This covers the **DEV_PLAN_2 player-first QoL
> phase**, which is being **parked mid-flight** (P2 built but not yet playtested/merged).
>
> **The new execution mode (the key shift).** After the proven T0→T5 sim shipped, the operator supplied
> `DEV_PLAN_2` — turn the sim into a *game*, **player-first, one QoL at a time**. The decisive change:
> **the gate became the human PLAYTEST (game-feel), not a headless assertion.** Cadence per phase: build
> + machine-verify (`dotnet build` + `dotnet test` + the **`t0–t5` headless scenarios as the regression
> net**), then raise a **`needs-human` playtest issue** on the team-pulse dashboard; the operator plays
> from their iPad and replies in the issue; the orchestrator **merges on "passes"** (fix-and-resubmit on
> fail). Same branch + PR-per-phase, self-merge on green machine-DoD — but **human-playtest-gated** rather
> than reviewer-gated.
>
> **Phases (all on the ColonyGame repo).** **P1a** (inventory, hand-gather, player-fed 3:1 coal Stone
> Smelter) — PASSED (#8), PR #7. **P1b** (cost-at-craft, Player Home auto-recruits the vagrant) — **FAILED
> first playtest (#10)** (leftover dev/sandbox keys leaked into the player build), resubmitted → passed,
> PR #9. **P1c** (Builder's Hut, C build-menu, fed blueprints) — passed *with notes* (#12), the
> keyboard-heavy/overlapping UX **triggered the operator's call for an overall UX review**, PR #11. **UX
> overhaul** (mouse cursor-picking, tabbed click-to-place build menu + cursor ghost, Player Home singleton
> / home-first / T1-only declutter) — PASSED, operator "really likes" it (#14), PR #13. **Build & grow
> flow** (auto build-jobs at nearest Builder's Hut, builder pulls from hut store, recruit-at-home, bounded
> + proximity deposits) — **THREE fix-and-resubmit rounds (#16)**, PR #15. **P2** (founder = citizen #0
> with light needs/health/respawn, the ascension chain, hand-gathered food node → Canteen feeding →
> recruit loop closes; `FounderTier`/`PlayerTier` reconciled, T5 win intact) — built, 234 tests, **PR #17
> OPEN, awaiting playtest (#18), NOT merged — in-flight when parked.**
>
> **Participating roles:** **orchestrator** (drove the run, branch/PR mechanics, machine-DoD verification,
> raised + closed the playtest issues, fix-and-resubmit routing), **senior-software-engineer** (the
> workhorse — every phase's sim + Godot presentation/UX layer + the playtest fix rounds), and the
> **quality-engineer lens** (the engineer wrote the tests; the `t0–t5` regression net was the discipline).
> The **operator was the gate** (human playtester). **reviewer-critic was much LESS engaged this phase** —
> the gate shifted from adversarial code-review to human playtest (a finding). **product-manager,
> software-architect, ux/ui-designer, security-engineer NOT engaged** — design was the operator's, this is
> a local game, and the UX direction came from the operator's own playtest feedback, not a UX phase.

> **Reading note:** the dominant lesson — **GUI/interaction cannot be headless-verified** — is **RECURRING
> within this phase** (the same bug *class* recurred across multiple playtests). It is the strongest
> candidate for a contract/build-DoD change.

## 1. Self-assessments

### Senior Software Engineer *(the workhorse this phase)*
- **Kept:** Shipped **~9 phase PRs in a day** (P1a, P1b+resubmit, P1c, the UX overhaul, the 3-round build
  & grow flow, then P2) — each riding on the **proven Godot-free sim core**, with the **`t0–t5` headless
  regression net never broken once** across the whole phase. The mouse-driven UX overhaul
  (cursor-picking, tabbed click-to-place menu, cursor ghost) and **founder-as-citizen-#0** (light needs,
  health/starvation/respawn, the ascension chain reconciling `FounderTier`/`PlayerTier` with the T5 win
  intact) are solid, well-tested work (200+ → **234** tests). FPS recovered to ~119.
- **Dropped / would change — the RECURRING ROOT, GUI cannot be headless-verified.** Every machine-DoD
  stayed **green while the game was broken in the hand**, because neither I nor the orchestrator (both
  headless) can click-test. A whole bug class surfaced *only* at the human playtest, repeatedly:
  - **Interactive controls rebuilt every frame** — the build menu rebuilt *every Button every frame*, so
    a click's press→release landed on a button destroyed mid-click; `Pressed` never fired. Number-keys
    masked it. **"Menu not clickable" was reported TWICE** before I found the root (#16 r1).
  - **UI enable-state not mirroring the sim's real preconditions** — the Recruit button looked enabled but
    silently no-op'd: the lone vagrant was unfed → avg-sat 0 → the sim refused, but the button never
    checked the satisfaction gate. A **lying, silently-no-op'ing button** (#16 r3).
  - **Input + movement buried inside a fixed-timestep sim loop** — the fixed-timestep refactor put input
    polling + movement inside the 20Hz loop and rendered the player with no interpolation, so at 119fps
    the avatar snapped 20×/s and **fast clicks were dropped** + movement was choppy (#16 r3).
  - **Pick / proximity / co-location** — couldn't click the Player Home because the vagrant rendered
    exactly on top of it; fixed by offsetting + resolving whatever is under the cursor (#16 r1).
  - **FPS regressions from per-frame work** — always-on-top `NoDepthTest` `Label3D` billboards on every
    node + building + per-frame string churn tanked FPS (#16 r2).
  - **Dev/sandbox affordances leaked into the player build** — leftover keys `B`/`1–4`/`R`/`F` placed
    free, unlabeled buildings; key `3` placed the *staffed* T1 smelter (not the player-fed Stone Smelter),
    producing the "built smelter, 100 ore + coal, no output" confusion (#10, the P1b first-playtest fail).
- **Lesson → notes.md (RECURRING within-phase):** none of these can be caught headless. I need a written
  **GUI interaction self-audit before requesting any playtest** — never rebuild interactive controls every
  frame (persist them, rebuild only on change); capture input **every frame**, never inside a
  fixed-timestep loop; **interpolate rendered positions** between sim ticks; avoid always-on-top /
  `NoDepthTest` labels and per-frame allocations at scale; **UI enable-state must mirror the sim's exact
  preconditions** (a silently-no-op'ing button is a bug); **never ship dev/sandbox affordances** in the
  player build. *(confidence: high)*

### Orchestrator
- **Kept:** Ran the new **playtest-gated** cadence cleanly: build + machine-verify each phase, raise a
  `needs-human` playtest issue on the dashboard, merge on operator "passes", **fix-and-resubmit on fail —
  no failing playtest was ever merged** (P1b failed once; build & grow flow took 3 rounds). The
  **dashboard `needs-human` loop is exactly how the operator interacts (from their iPad)** and proved its
  design across ~6 playtest issues. Routed deferred feedback to later phases correctly (fuel-units→P3,
  F1→P5, then pulled F1 forward when it mattered). Stopped exactly at "P2 built, awaiting playtest" when
  parked.
- **Dropped / would change:** The **machine-DoD is necessary-but-insufficient** for player-facing work,
  and I had no instrument to catch the GUI bug class before the operator did — so the **iteration count
  was high** (build & grow flow = 3 playtest rounds, P1b = 2, and "menu not clickable" recurred twice).
  Each round costs an operator round-trip. I treated a green machine-DoD as "ready to playtest" when for
  interactive work it only means "ready to *attempt* a playtest."
- **Lesson → notes.md:** for player-facing phases, green machine-DoD ≠ playtest-ready — gate the playtest
  request behind a **GUI interaction self-audit** (engineer) and a **pre-playtest interaction review**
  (reviewer reads the GUI diff for the known pitfalls), to cut the operator round-trip count. The
  dashboard `needs-human` loop is the right mechanism; expect iteration on game-feel. *(confidence: high)*

### Quality Engineer *(lens — the `t0–t5` regression net)*
- **Kept:** The **`t0–t5` headless scenarios did their job perfectly as the regression net** — across ~9
  PRs that heavily refactored the presentation layer (including a fixed-timestep refactor and the
  `FounderTier`/`PlayerTier` reconciliation), the proven sim economy **never accidentally regressed once**.
  Test count grew 200+ → 234. **Zero escaped *sim* defects.**
- **Dropped / would change:** The entire bug surface this phase was the **UI/interaction layer the test
  net by design does not cover.** Every playtest failure was a presentation/interaction defect (clickable
  buttons, enable-state, input coupling, FPS, dev keys) — invisible to `dotnet test` and the headless
  scenarios. The machine layer was a false "all green."
- **Lesson → notes.md:** the headless test net verifies the **sim**; it is structurally blind to the
  **feel/UI** layer. For player-facing phases the only real gate is human, and the test net should be read
  as "the sim didn't regress" — **not** "the game works." *(confidence: high)*

### Reviewer / Critic *(thinned out this phase — a finding)*
- **Kept:** N/A for most of the phase — with the gate shifted to human playtest, the adversarial
  code-review role was **largely not exercised** (a deliberate consequence of the mode, not a failure).
- **Dropped / would change:** The GUI bug class that drove every playtest failure (rebuild-every-frame,
  enable-state-mismatch, input-coupled-to-sim-loop, dev-keys-in-player-build) is **exactly the kind of
  static, diff-readable defect a pre-playtest review could catch** — and the operator caught them instead,
  at a higher cost. The role had spare capacity precisely when a different flavour of review would have
  paid off most.
- **Lesson → notes.md:** in player-facing phases where the gate is human playtest, **pivot from
  adversarial code-review to a GUI-pitfall pre-playtest review** — read the GUI diff for the known
  un-headless-verifiable classes (controls rebuilt per frame; UI enable-state not mirroring sim
  preconditions; input/movement buried in a fixed-timestep loop; per-frame allocations / always-on-top
  draws; dev affordances in the player build) **before** the operator burns a round-trip. *(confidence: high)*

### Roles NOT engaged (and why)
- **Product Manager / Software Architect / UX-UI Designer:** no PRD, architecture, or UX phase — the
  design and `DEV_PLAN_2` were the operator's, and the UX direction came from the **operator's own
  playtest feedback** (the #12 call for a UX review → the overhaul), not a designer. The "STOP if it isn't
  fun" / playtest gate replaced the design lifecycle.
- **Security Engineer:** Hiraeth remains a local, single-player Godot game with no network/auth/deploy
  surface — the deployed-web-service security class does not apply.

## 2. 360 reviews (chain neighbors)

### Orchestrator → Senior Software Engineer
- **Worked well:** The decoupled sim core let you turn out ~9 UX PRs in a day with the regression net
  intact — that velocity is only possible because the sim is sacred and Godot-free. Fix turnarounds across
  the 3-round build/grow flow were fast and complete.
- **Friction / suggestion:** The recurring root is that **interactive UI can't be machine-verified** — and
  "menu not clickable" reached the operator **twice**. Run a written **GUI interaction self-audit** (the
  rebuild-every-frame / enable-state / input-coupling / FPS / dev-keys checklist) before you tell me a
  phase is playtest-ready, so we stop spending operator round-trips on diff-visible bugs.

### Senior Software Engineer → Orchestrator
- **Worked well:** The `needs-human` dashboard playtest loop is the right mechanism and matches exactly how
  the operator works (iPad, reply-in-issue); fix-and-resubmit discipline (never merge a failing playtest)
  kept quality honest through the rounds.
- **Friction / suggestion:** Treating green machine-DoD as "playtest-ready" set the bar too low for
  interactive work — add a pre-playtest interaction gate so the high round count (3 on build/grow flow)
  comes down.

### Orchestrator → Reviewer / Critic *(on the thinned role)*
- **Worked well:** Correctly stepped back when the gate became human playtest — no redundant adversarial
  code-review overhead.
- **Friction / suggestion:** There was a **review-shaped hole** the whole phase: the GUI-pitfall classes
  the operator kept catching are static and diff-readable. Pivot to a **pre-playtest GUI interaction
  review** in player-facing phases — that's where your adversarial eye now earns its keep.

### Quality lens → the phase (process self-review)
- **Worked well:** The `t0–t5` regression net is the unsung hero — it let a heavy presentation refactor
  proceed with confidence the proven economy held; 0 escaped sim defects.
- **Friction / suggestion:** The test net's green light was **misread as product-works** when it only ever
  meant **sim-didn't-regress**. The feel/UI gate is entirely human; name that explicitly so a green CI is
  never mistaken for a passing game.

### Orchestrator + Engineer → the playtest-gated MODE (process self-review)
- **Worked well:** Replacing the headless assertion with a **human playtest gate** + the dashboard
  `needs-human` loop is the correct adaptation for player-first work, and the **decoupled sim core +
  `t0–t5` net** made it safe to iterate fast on the surface. The mode proved its design.
- **Friction / suggestion:** The mode's **weakness is the gap between green machine-DoD and the game
  working in the hand** — an entire un-headless-verifiable bug class lives there, with a high iteration
  cost. The mode needs a **pre-playtest interaction gate** (engineer self-audit + reviewer GUI review)
  written into the build-DoD so the next player-facing run starts from the proven bar.

## 3. Improvement recommendations

| # | Recommendation | Target role | Level (notes / contract / skill) | Severity | Owner |
|---|----------------|-------------|----------------------------------|----------|-------|
| 1 | **GUI interaction self-audit before requesting any playtest (the big one).** A written checklist run before a phase is declared playtest-ready: (a) no interactive control rebuilt every frame — persist, rebuild only on change; (b) input captured **every frame**, never inside a fixed-timestep loop; (c) rendered positions **interpolated** between sim ticks; (d) no always-on-top/`NoDepthTest` labels or per-frame allocations at scale; (e) **UI enable-state mirrors the sim's exact preconditions** (a silently-no-op'ing button is a bug); (f) **no dev/sandbox affordances in the player build**. RECURRING within-phase root (clickable-menu reported twice; lying recruit button; dropped clicks; dev keys). **Candidate contract / build-DoD** → **operator approval**. | senior-software-engineer (+ gates.md / mode DoD) | notes (done); **candidate contract / build-DoD** | high | Senior Eng |
| 2 | **Pre-playtest interaction review — reviewer reads the GUI diff for the known un-headless-verifiable pitfalls** (rebuild-every-frame; enable-state mismatch; input/movement coupled to the sim loop; per-frame allocs / always-on-top draws; dev affordances in player build) **before** the operator burns a round-trip. Pivots the thinned reviewer role to where it now pays off; aims to cut the high iteration count (build/grow flow = 3 rounds). **Candidate process / contract** → **operator approval**. | reviewer-critic / orchestrator (+ gates.md / mode DoD) | notes (done); **candidate process / contract** | high | Reviewer / Orchestrator |
| 3 | **Name the gate explicitly: machine-DoD = "the sim didn't regress", NOT "the game works."** For player-facing work the human playtest is **THE** gate; green `dotnet test` + green `t0–t5` is necessary-but-insufficient. Read green CI accordingly; expect game-feel iteration. | orchestrator / quality-engineer | notes (done) | high | Orchestrator / QA |
| 4 | **Reinforce: the decoupled, Godot-free sim core + the `t0–t5` headless regression net held up beautifully** — ~9 presentation PRs (incl. a fixed-timestep refactor + the `FounderTier`/`PlayerTier` reconciliation) rode on it with **0 accidental sim regressions**. Keep the sacred-sim seam + regression net as the pattern for any UI-heavy autonomous build. | senior-software-engineer | notes (done) — positive signal | — | Process Eng |
| 5 | **Reinforce: the dashboard `needs-human` playtest loop is the right operator-interaction mechanism** — the operator playtested ~6 phases from their iPad, replied in-issue, and the orchestrator merged on "passes" / fixed-and-resubmitted on fail. Keep it as the human-gate for player-facing work; **fix-and-resubmit discipline (never merge a failing playtest) held throughout.** | orchestrator | notes (done) — positive signal | — | Process Eng |
| 6 | **Reinforce: the reviewer thinned out when the gate became playtest** — a real signal that the adversarial code-review role should *pivot* (rec #2) rather than persist unchanged in player-facing phases. Captured so future phase-mode runs deploy the reviewer where it's load-bearing. | reviewer-critic / process-engineer | **process observation; informs rec #2** | — | Process Eng |

## 4. Actions taken

| Recommendation | Result | Link |
|----------------|--------|------|
| #1 | Notes appended to senior-software-engineer (RECURRING within-phase root: GUI can't be headless-verified + the 6-point interaction self-audit); flagged as **candidate contract / build-DoD** → route to operator approval (no persona.md edit) | `.claude/agents/senior-software-engineer/notes.md` |
| #2 | Notes appended to reviewer-critic (pivot to pre-playtest GUI interaction review in player-facing phases); flagged as **candidate process / contract** → operator approval | `.claude/agents/reviewer-critic/notes.md` |
| #3 | Notes appended to orchestrator (machine-DoD necessary-but-insufficient; human playtest is THE gate; expect iteration) and quality-engineer (test net = sim-didn't-regress, not game-works) | `.claude/agents/orchestrator/notes.md`, `.claude/agents/quality-engineer/notes.md` |
| #4 | Recorded as positive signal; note appended to senior-software-engineer (sim seam + `t0–t5` net held through ~9 UX PRs) | `.claude/agents/senior-software-engineer/notes.md` |
| #5 | Recorded as positive signal; note appended to orchestrator (dashboard `needs-human` loop + fix-and-resubmit discipline) | `.claude/agents/orchestrator/notes.md` |
| #6 | Recorded as a process observation informing rec #2; note appended to reviewer-critic | `.claude/agents/reviewer-critic/notes.md` |

### Routed follow-ups — ⏳ AWAITING OPERATOR APPROVAL

Per DESIGN §5, contract/build-DoD/process changes need the human side of the gate. **No `persona.md` was
edited.** The following are flagged as candidates for operator approval:

- ⏳ **GUI interaction self-audit as a build-DoD step (rec #1)** for player-facing / game phases: a written
  6-point checklist (controls not rebuilt per frame; input captured every frame not in a fixed-timestep
  loop; rendered positions interpolated between ticks; no always-on-top/`NoDepthTest` labels or per-frame
  allocs at scale; UI enable-state mirrors the sim's exact preconditions; no dev/sandbox affordances in the
  player build) run **before requesting a playtest**. Candidate for the player-facing build-DoD in
  `docs/gates.md` and/or `senior-software-engineer/persona.md`.
- ⏳ **Pre-playtest interaction review (rec #2):** in player-facing phases the reviewer-critic **pivots**
  from adversarial code-review to a **GUI-pitfall diff review** against the same checklist, run before the
  orchestrator raises the `needs-human` playtest issue. Candidate for `docs/gates.md` (the playtest-gated
  mode) and/or `reviewer-critic/persona.md`.

## 5. Metrics (signal)

- **Phases / PRs:** **5 phase PRs merged** — P1a (#7), P1b (#9, after 1 resubmit), P1c (#11), UX overhaul
  (#13), build & grow flow (#15, after 3 rounds). **P2 (PR #17) built + machine-verified but OPEN /
  unmerged — in-flight when parked**, awaiting playtest #18. (~9 distinct build/resubmit cycles total.)
- **Playtest gate:** **6 `needs-human` playtest issues** raised on the dashboard (#8, #10, #12, #14, #16,
  #18) — the operator played each from their iPad and replied in-issue.
- **Playtest outcomes:** P1a **pass** (#8); P1b **FAIL → resubmit → pass** (#10); P1c **pass-with-notes**
  → triggered the UX overhaul (#12); UX overhaul **pass**, "really likes it" (#14); build & grow flow
  **3 fix-and-resubmit rounds** before pass (#16); P2 **awaiting** (#18). **No failing playtest was ever
  merged.**
- **Test growth:** 200+ → **234** sim tests, all green every phase; **`t0–t5` headless scenarios green
  throughout** (the regression net never broke).
- **Escaped *sim* defects: 0** — the decoupled sim core never accidentally regressed across ~9
  presentation-heavy PRs (incl. a fixed-timestep refactor + `FounderTier`/`PlayerTier` reconciliation).
- **UI iteration count: HIGH** — build & grow flow alone took **3 playtest rounds**; P1b took 2; "menu not
  clickable" was reported **twice**. Every playtest failure was a UI/interaction defect.
- **Recurring root (within-phase):** **GUI/interaction cannot be headless-verified** — the machine-DoD
  stayed green while the game was broken in the hand. Bug classes seen repeatedly: interactive controls
  rebuilt every frame (clicks lost); UI enable-state not mirroring sim preconditions (the lying recruit
  button); input/movement buried in a fixed-timestep loop (dropped clicks + choppy movement); pick /
  proximity / co-location; FPS regressions from per-frame allocs / always-on-top draws; dev/sandbox
  affordances leaking into the player build.
- **Positive signals:** the **decoupled, Godot-free sim core + `t0–t5` regression net** made fast UX
  iteration safe (0 sim regressions); the **dashboard `needs-human` playtest loop** proved it is exactly
  how the operator interacts; **fix-and-resubmit discipline** held; FPS recovered to ~119; the mouse-driven
  UX overhaul and founder-as-citizen-#0 are solid. The **reviewer-critic thinned out** when the gate
  shifted to playtest — a signal to *pivot* the role (rec #2), not retire it.
- **State at parking:** P2 mechanism complete + tested (234 tests; T5 win intact); **one descope flagged**
  by the engineer — the *interactive research-start* UI (needs moving the Research-Lab T3→T1, which would
  perturb the proven t3 economy), deferred to a follow-up. PR #17 / playtest #18 left open for whenever the
  phase resumes.
</content>
</invoke>
