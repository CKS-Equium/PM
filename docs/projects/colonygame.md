---
slug: colonygame
repo: https://github.com/CKS-Equium/ColonyGame
status: parked
phase: build (DEV_PLAN_2 player-first QoL, paused mid-flight)
created: 2026-06-07
team: [orchestrator, software-architect, senior-software-engineer, quality-engineer, reviewer-critic]
board: n/a
---

## Summary

**Hiraeth** — a space-themed grand-strategy / colony-simulation hybrid in Godot 4 + C#. A living
economy where citizens are simultaneously labour, consumers, and the progression gate; a curated
materials tree forces outward expansion. Built per the project's own `docs/DEV_PLAN.md` (single-plane
T0→T5 gate ladder, placeholder primitives, headless-verified). Run via the PM team's PR conventions.

## Current state

**Single-plane DEV_PLAN T0→T5 COMPLETE (2026-06-08, overnight autonomous run) — awaiting operator
playtest.** All six gates built, reviewed, and merged to `main` (PRs #1–#6, each self-merged on
green DoD; `gate/t0…t5` branches kept as checkpoints). **132 sim unit tests green; all six headless
gate scenarios pass (`[GATE t0..t5 PASS]`); `dotnet build ColonyGame.sln` clean; `src/Sim` stays
Godot-free.** The win condition fires (tier-5-of-everything → Tier-6 Emperor ascension at the
verified tick) and the sandbox continues past victory.

**What is verified vs. what needs the human:** the *systems* (economy, materials/property-tag
recipes, population spine, power, research, circuits, builder-crew pyramid, logistics web, the five
needs, the win condition) are unit-tested + headless-verified. The *feel/visuals/fun* and the
interactive GUI render are the operator's playtest (the one DoD item an agent can't clear — the
headless scene boots clean, but GUI rendering/input wasn't run). Presentation is deliberate
placeholder primitives (cylinders/cuboids/markers) per the dev plan's art bar.

## PARKED — 2026-06-09

Project **parked mid-flight** after the DEV_PLAN_2 *player-first QoL* phase. The proven T0→T5 sim
(above) plus the playable bootstrap arc, the mouse-driven UX overhaul, and the build-&-grow flow are
**merged to `main`** (PRs #7/#9/#11/#13/#15). **P2** (founder = citizen #0 + the food node + the
ascension chain) is **built + machine-verified (234 tests, t0–t5 green) but NOT playtested/merged** —
**PR #17 left open**, playtest **issue #18 open**, as the clean resume point. Phase retrospective:
[docs/postmortems/colonygame-dev-plan-2.md](../postmortems/colonygame-dev-plan-2.md).

**Resume here:** playtest P2 (#18) → merge or fix; then the open question (Research-Lab T3→T1 to make
the ascension research-start clickable) and P3–P6 (feedback polish, build-mode polish, camera,
day/night). Central lesson of the phase: **GUI/interaction can't be headless-verified** — the
human playtest is the real gate (see the retrospective + the routed contract candidates).

## Decision log

- 2026-06-07 — Adopted the project's `docs/DEV_PLAN.md` (single-plane economic build, T0→T5).
  Operator directive: use the PM team's PR rules (not the plan's solo/no-PR mode); **do not stop at
  internal human gates** — run until plan-complete-needs-playtest or a hard blocker; branch per gate
  as a checkpoint. Human "playable to its tier" check is deferred to the operator; headless gate
  scenario is the machine proxy.
