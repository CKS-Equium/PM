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

**Registered for dashboard visibility; the overnight autonomous build is QUEUED, not started**
(awaiting the operator's "go"). When it runs, work proceeds on `gate/tN` branches with a PR per gate
(self-merged on green DoD: `dotnet build` + sim `dotnet test` + headless gate scenario + Godot
headless import + the `using Godot` grep guard). Activity (gate PRs) will stream here once it starts.

## Decision log

- 2026-06-07 — Adopted the project's `docs/DEV_PLAN.md` (single-plane economic build, T0→T5).
  Operator directive: use the PM team's PR rules (not the plan's solo/no-PR mode); **do not stop at
  internal human gates** — run until plan-complete-needs-playtest or a hard blocker; branch per gate
  as a checkpoint. Human "playable to its tier" check is deferred to the operator; headless gate
  scenario is the machine proxy.
