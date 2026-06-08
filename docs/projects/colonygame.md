---
slug: colonygame
repo: https://github.com/CKS-Equium/ColonyGame
status: active
phase: build
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

## Decision log

- 2026-06-07 — Adopted the project's `docs/DEV_PLAN.md` (single-plane economic build, T0→T5).
  Operator directive: use the PM team's PR rules (not the plan's solo/no-PR mode); **do not stop at
  internal human gates** — run until plan-complete-needs-playtest or a hard blocker; branch per gate
  as a checkpoint. Human "playable to its tier" check is deferred to the operator; headless gate
  scenario is the machine proxy.
