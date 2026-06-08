# Post-mortem: colonygame ("Hiraeth")

- **Project:** colonygame (Hiraeth) · **Repo:** https://github.com/CKS-Equium/ColonyGame
- **Dates:** 2026-06-07 → 2026-06-08 (single-plane DEV_PLAN T0→T5 complete; awaiting operator playtest)
- **Facilitator:** Process Engineer
- **Outcome:** Plan-complete-needs-playtest — the team's **third** project, and its **first execution-mode run**

> The operator's **own** Godot 4 + C# colony-sim, built **per the project's `docs/DEV_PLAN.md`** in
> one overnight autonomous run. This was a **new execution MODE**, not the standard lifecycle: the
> team did **no discovery or design** (the design and dev plan were the operator's), and the project's
> per-gate **Definition of Done** — `dotnet build ColonyGame.sln` (full solution incl. the Godot
> project) + sim `dotnet test` + a headless `--scenario=tN` runner that asserts end-state and prints
> `[GATE tN PASS]`/exit code + Godot `--headless --import` + a `grep "using Godot" src/Sim`
> architecture guard — **replaced the PM 7-gate lifecycle**. The gate ladder was the dev plan's own
> **T0→T5**. Operator directives: use PM **PR conventions** (branch `gate/tN` + one PR per gate,
> **self-merged on green DoD**), **do NOT stop at internal human gates**, run until
> plan-complete-needs-playtest or a hard blocker. A reviewer **BLOCK** was treated as
> **fix-before-merge, not a human stop**.
>
> **Participating roles:** orchestrator (drove the run, git/PR mechanics + independent per-gate DoD
> verification), **senior-software-engineer** (the workhorse — every gate's sim core + Godot layer +
> scenario), **reviewer-critic** (adversarial review per gate), and the **quality-engineer lens**
> (the engineer wrote the tests; the QA discipline was applied through the reviewer). **product-manager,
> software-architect, ux/ui-designer, and security-engineer were NOT engaged** — there was no
> PRD/architecture/UX phase (design was the operator's), and this is a local game, not a deployed web
> service with a network attack surface.

> **Reading note:** lessons marked **RECURRING** also appeared in the team-pulse-dashboard and/or
> hearthflix post-mortems — they are the strongest candidates for contract/skill promotion.

## 1. Self-assessments

### Orchestrator
- **Kept:** Drove the whole overnight run end-to-end under the new mode — branch-per-gate, one PR per
  gate, **self-merge on green DoD**, and **independent DoD verification** at each gate (didn't take the
  engineer's word). Honoured the operator directive to **not stop at internal human gates** and to
  treat a reviewer BLOCK as **fix-before-merge**; the run completed all six gates and stopped exactly
  at plan-complete-needs-playtest, as instructed. **0 escaped defects to `main`.**
- **Dropped / would change:** Two mechanical slips, both Godot-specific. (1) **Full-solution-build
  gap** — at T2 the engineer built only the sim assembly, so a `Vec2` `using` error in the Godot
  project slipped through and the old binary ran "unknown scenario"; the DoD's "scene imports & runs"
  was not actually proven until I mandated `dotnet build ColonyGame.sln` (the **whole** solution) per
  gate. (2) **Git friction** — Godot regenerates `.uid`/`.godot`/`.import` artifacts on every headless
  run, which blocked the post-merge local-`main` fast-forward and forced a `stash -u → ff → drop` dance
  ×6.
- **Lesson → notes.md:** The per-gate build-DoD must build the **full solution**, not just the sim,
  or the "scene imports & runs" guarantee is hollow; a Godot+C# project in an autonomous loop needs a
  defined post-merge sync recipe and an upfront `.uid`/generated-artifact commit policy; **BLOCK =
  fix-before-merge (not human stop) worked** — quality stayed high without a single human stop.
  *(confidence: high)*

### Senior Software Engineer *(the workhorse this run)*
- **Kept:** Built **every** gate end-to-end — pure-C# sim core, Godot presentation layer, and the
  headless acceptance scenario — across all six gates, growing the suite 17→39→66→89→115→**132** green
  tests. The **Godot-free `src/Sim` seam** (command-in/state-out, no `using Godot`) is what made an
  otherwise-unverifiable game **autonomously buildable and testable**; the `--scenario=tN` headless
  runner is the machine proxy for "is it playable."
- **Dropped / would change:** **RECURRING ROOT — authored-data correctness was unguarded.** At T1,
  mine-vs-smelter was **inferred from an overloaded `reach` field**, so a mis-authored `reach` made the
  smelter silently never smelt — the economy didn't close, and it was caught **only by the headless
  scenario, not the 38 unit tests** (which ran a hand-authored fixture, not the real
  `data/buildings.json`). The same class recurred: T2 distinct-units (untested), T4 warehouses were a
  **one-way "roach motel"** (couriers deposited goods nothing read back, silently draining production
  inputs into dead storage), masked by a scenario that warehoused a throwaway good and tests that
  asserted only conservation, never *usability*. Also the T2 build-only-sim slip (signal #4).
- **Lesson → notes.md:** Prefer an **explicit discriminator** (`ProductionKind`) over inference from an
  incidental field, and **validate authored data at load time (throw on bad data)**; acceptance
  scenarios must exercise the **full produce→consume loop** and assert *usability*, not mere
  presence/conservation; verify the **full-solution build** every gate, not just the sim assembly.
  *(confidence: high)*

### Quality Engineer *(lens — coverage was authored by the engineer, QA discipline applied via the reviewer)*
- **Kept:** The suite that did exist had real teeth at the unit level (mining-rate, labour-gating,
  power gating, starvation, housing cap), and the headless scenarios were a strong acceptance layer
  that caught the T1 economy break unit tests missed.
- **Dropped / would change:** The coverage had a **systematic blind spot for a data-driven design**:
  unit tests ran against **hand-authored fixtures**, not the **real shipped `data/buildings.json`**, so
  authored-data breaks were invisible to tests (T1). And conservation-only assertions let the T4
  warehouse stranding pass — the goods were *conserved* (in dead storage) but never *usable*. The T2
  property-tag core guarantee (two same-property inputs draw **distinct** units, no double-spend) was
  **untested**, and the T1 scenario's satisfaction assertion had been silently passing at `avgSat=0`.
- **Lesson → notes.md:** For a data-driven system, ship a **real-shipped-data integration test every
  gate from the start** (run the real `data/*.json`, not a fixture); assert **usability** (goods can be
  produced *and consumed*), not just presence/conservation; tighten assertions so they can't pass at a
  degenerate value (`avgSat=0`). *(confidence: high)*

### Reviewer / Critic *(earned its keep AGAIN)*
- **Kept:** Adversarial review every gate caught **two real "passes-the-gate-breaks-the-game" bugs**
  that green DoD + passing scenarios hid — **T1** (smelter never smelted; root: unguarded authored
  data) and **T4** (warehouse one-way roach motel) — plus the **T2 distinct-units MAJOR test-gap** and
  the T2 recreation-applied-to-pre-T2-colonies design issue. Both BLOCKs were turned around to clean
  re-review **before merge** with regression tests of **proven teeth** (T1: flip the kind → red; T4:
  15 stranded without the fix → 0 with). This is the **third consecutive project** the review gate
  caught release-relevant defects tests missed — and it is **most** load-bearing in an autonomous run
  where no human watches each step.
- **Lesson → notes.md (first entry for this role):** On autonomous/unattended runs, stay adversarial
  on the **gap between "green DoD" and "the game/product actually works"** — green build + passing
  acceptance scenario does **not** mean usable; hunt for state that is produced/stored but never
  read/consumed, and for tests that assert conservation/presence rather than end-use. *(confidence: high)*

### Roles NOT engaged (and why)
- **Product Manager / Software Architect / UX-UI Designer:** no PRD, architecture, or UX phase — the
  **design and the `docs/DEV_PLAN.md` were the operator's**, and the directive was to *execute*, not
  discover/design. The dev-plan gate ladder (T0→T5) and per-gate DoD replaced the PM lifecycle gates.
- **Security Engineer:** Hiraeth is a **local, single-player Godot game** with no network/auth/deploy
  surface — none of the deployed-web-service security class (headers/CSP, CVE deps, container, bind)
  that drove the team-pulse/hearthflix security findings applies here. The `grep "using Godot" src/Sim`
  architecture guard was the only "non-functional invariant" gate, and the engineer/orchestrator owned it.

## 2. 360 reviews (chain neighbors)

### Orchestrator → Senior Software Engineer
- **Worked well:** The Godot-free `src/Sim` seam + the `--scenario=tN` headless runner made the run
  *possible* — without them an autonomous agent could not verify a game. End-to-end builds (T2–T5) were
  fast.
- **Friction / suggestion:** The T2 "build only the sim" slip meant a Godot-project compile error
  reached the gate; **build the full solution** as part of your own DoD before requesting review, so
  "scene imports & runs" is actually proven.

### Senior Software Engineer → Orchestrator
- **Worked well:** Independent per-gate DoD verification and the BLOCK-as-fix-before-merge call kept the
  bar high without ever stopping the run; the branch-per-gate checkpoints were a good safety net.
- **Friction / suggestion:** The Godot generated-artifact churn (`.uid`/`.godot`/`.import`) needs a
  defined commit/ignore policy and a post-merge sync recipe up front, rather than a manual `stash` dance
  every gate.

### Reviewer / Critic → Senior Software Engineer
- **Worked well:** Fix turnarounds were fast and complete, and the regression tests added on each BLOCK
  had **proven teeth** (demonstrated red-without-fix), which is exactly right.
- **Friction / suggestion:** The recurring root is **authored-data correctness** — discriminate roles
  explicitly and validate data at load; and make scenarios exercise the **full produce→consume loop** so
  "stored but never consumed" defects (warehouse) can't pass an acceptance run.

### Senior Software Engineer → Reviewer / Critic
- **Worked well:** Caught the two bugs (T1, T4) that green DoD + passing scenarios genuinely hid, and the
  T2 distinct-units gap I'd have shipped — the adversarial lens is the reason 0 defects escaped to `main`.
- **Friction / suggestion:** None — the BLOCK-before-merge loop is the right shape for unattended runs.

### Reviewer / Critic → Quality lens (the test coverage)
- **Worked well:** Strong unit-level coverage and a real headless acceptance layer that caught the T1
  economy break.
- **Friction / suggestion:** Fixture-only unit tests + conservation-only assertions are a blind spot for
  a data-driven economy — a **real-shipped-data integration test every gate** and **usability assertions**
  would have caught T1/T4 at test time instead of at review.

### Orchestrator + Reviewer / Critic → the new execution MODE (process self-review)
- **Worked well:** Replacing the 7-gate lifecycle with the project's per-gate DoD + reviewer-as-gate was
  the right adaptation for an operator-designed, no-discovery build; the run shipped six clean gates
  overnight with 0 escaped defects.
- **Friction / suggestion:** The mode needs its **build-DoD written down** (full-solution build;
  real-shipped-data integration test; produce→consume usability assertion) so the next execution-mode run
  starts from the proven bar rather than rediscovering it gate-by-gate.

## 3. Improvement recommendations

| # | Recommendation | Target role | Level (notes / contract / skill) | Severity | Owner |
|---|----------------|-------------|----------------------------------|----------|-------|
| 1 | **Guard authored-data correctness — explicit discriminators + load-time validation that throws on bad data** (vs inferring kind from an incidental field). For data-driven designs this is the strongest pattern; surfaced as the RECURRING ROOT across T1/T2/T3/T4. **Candidate contract / build-DoD** change → **operator approval**. | senior-software-engineer (+ gates.md / mode DoD) | notes (done); **candidate contract / build-DoD** | high | Senior Eng |
| 2 | **Acceptance scenarios must exercise the full produce→consume loop and assert *usability*, not presence/conservation** — the T4 warehouse "roach motel" passed a conservation-only scenario. Also avoid degenerate-passing assertions (`avgSat=0`). | senior-software-engineer / quality-engineer | notes (done) | high | Senior Eng / QA |
| 3 | **Ship a real-shipped-data integration test every gate from the start** — run the real `data/*.json`, not a hand-authored fixture, so authored-data breaks (T1) are caught at test time. **Candidate build-DoD** for data-driven projects → **operator approval**. | quality-engineer (+ gates.md / mode DoD) | notes (done); **candidate build-DoD** | high | QA |
| 4 | **Verify the FULL-solution build every gate (`dotnet build <Solution>.sln`), not just the sim assembly** — the T2 `Vec2` Godot-project error proved that a sim-only build leaves "scene imports & runs" unproven. **Candidate build-DoD** → **operator approval**. | senior-software-engineer / orchestrator (+ mode DoD) | notes (done); **candidate build-DoD** | high | Senior Eng / Orchestrator |
| 5 | **Define a Godot+C# autonomous-loop git recipe** — commit policy for generated `.uid`/`.godot`/`.import` artifacts and a post-merge sync recipe, to remove the per-gate `stash -u → ff → drop` dance. | orchestrator / devops-release-engineer | notes (done); **skill/process** | med | Orchestrator |
| 6 | **Document the new execution MODE** — operator-designed, no-discovery "execute the dev plan" runs where the project's per-gate DoD + reviewer-as-gate replace the 7-gate lifecycle, BLOCK = fix-before-merge, and the run stops at plan-complete-needs-playtest. Capture so the next such run starts from the proven bar. | process-engineer / orchestrator (+ DESIGN/gates.md) | **process decision; document** | med | Process Eng |
| 7 | **Reinforce: the pure-(non-Godot) sim seam + headless-scenario-as-acceptance pattern made an unverifiable game autonomously buildable** — strong positive signal; the `--scenario=tN` runner is the machine proxy for the human "is it playable" gate and caught the T1 bug unit tests missed. Keep this seam as the pattern for any game/UI-heavy autonomous build. | senior-software-engineer | notes (done) — positive signal | — | Process Eng |
| 8 | **Reinforce: the review gate earns its keep — THIRD consecutive project** — caught the T1/T4 "passes-the-gate-breaks-the-game" bugs and the T2 test-gap that green DoD + passing scenarios hid. Especially load-bearing on unattended runs. Keep adversarial-on-autonomous; BLOCK = fix-before-merge worked. | reviewer-critic | notes (done) — positive signal | — | Process Eng |

## 4. Actions taken

| Recommendation | Result | Link |
|----------------|--------|------|
| #1 | Notes appended to senior-software-engineer (RECURRING ROOT); flagged as **candidate contract / build-DoD** change → route to operator approval (not applied to persona.md) | `.claude/agents/senior-software-engineer/notes.md` |
| #2 | Notes appended to senior-software-engineer and quality-engineer (usability-not-conservation) | `.claude/agents/senior-software-engineer/notes.md`, `.claude/agents/quality-engineer/notes.md` |
| #3 | Notes appended to quality-engineer; flagged as **candidate build-DoD** for data-driven projects → route to operator approval | `.claude/agents/quality-engineer/notes.md` |
| #4 | Notes appended to senior-software-engineer and orchestrator; flagged as **candidate build-DoD** → route to operator approval | `.claude/agents/senior-software-engineer/notes.md`, `.claude/agents/orchestrator/notes.md` |
| #5 | Notes appended to orchestrator (Godot `.uid`/post-merge-sync recipe) | `.claude/agents/orchestrator/notes.md` |
| #6 | Recorded as a process decision; execution-MODE documentation candidate for DESIGN/gates.md — **escalated** to operator | this report |
| #7 | Recorded as positive signal; note appended to senior-software-engineer | `.claude/agents/senior-software-engineer/notes.md` |
| #8 | Recorded as positive signal; note appended to reviewer-critic (first entry for this role) | `.claude/agents/reviewer-critic/notes.md` |

### Routed follow-ups — ⏳ AWAITING OPERATOR APPROVAL

Per DESIGN §5, contract/build-DoD changes need the human side of the gate. **No `persona.md` was edited.**
The following are flagged as candidates for operator approval:

- ⏳ **Build-DoD (rec #1, #3, #4) for data-driven / game projects:** (a) explicit discriminators +
  load-time validation that throws on bad data; (b) a **real-shipped-data integration test every gate**;
  (c) **full-solution build** (`dotnet build *.sln`) every gate. Candidate for a data-driven/game build-DoD
  in `docs/gates.md` and/or `senior-software-engineer/persona.md` + `quality-engineer/persona.md`.
- ⏳ **Execution-MODE documentation (rec #6):** capture the operator-designed "execute the dev plan" mode
  (per-gate project DoD + reviewer-as-gate replace the 7-gate lifecycle; BLOCK = fix-before-merge; stop at
  plan-complete-needs-playtest) in DESIGN.md / gates.md as a first-class alternative lifecycle.

## 5. Metrics (signal)

- **Gates / PRs:** 6 dev-plan gates **T0→T5**, one PR each (**#1–#6**), all merged to `main`;
  `gate/t0…t5` branches kept as checkpoints.
- **Test growth:** 17 → 39 → 66 → 89 → 115 → **132** sim unit tests, all green. All six headless gate
  scenarios pass (`[GATE t0..t5 PASS]`, exit 0).
- **Reviewer outcomes:** **2 BLOCK** (T1, T4) + **4 PASS** (T0, T2*, T3, T5; *T2 PASS carried a fixed
  MAJOR test-gap) → the review gate **found release-relevant issues in 3 of 6 gates**.
- **Escaped defects to `main`: 0** — every issue caught pre-merge by the reviewer or the headless
  scenario.
- **Bugs the headless scenario caught that unit tests missed:** T1 (smelter never smelted — economy
  didn't close).
- **Bugs the reviewer caught that both tests and the scenario missed/masked:** T4 warehouse stranding
  (conservation-only scenario), T2 distinct-units gap (untested core guarantee).
- **Notable rework loops:** 2 (T1 BLOCK, T4 BLOCK), both fix-before-merge with proven-teeth regression
  tests; no discovery/design re-round (no discovery/design phase).
- **Recurring lessons:** 1 dominant — **authored-data correctness was unguarded** (the RECURRING ROOT
  across T1/T2/T3/T4); the **review gate earns its keep** signal also recurred (third consecutive
  project).
- **Positive signals:** the **Godot-free sim seam + headless-scenario-as-acceptance** pattern made an
  otherwise-unverifiable game autonomously buildable; the **review gate** caught the
  passes-the-gate-breaks-the-game bugs on an unattended run; **BLOCK = fix-before-merge** held quality
  high with **zero human stops**.
- **Remaining DoD item for the human:** the GUI render + whether it's *fun* (operator playtest) — the
  one DoD item an agent can't clear; the headless scene boots clean but GUI rendering/input wasn't run.
</content>
</invoke>
