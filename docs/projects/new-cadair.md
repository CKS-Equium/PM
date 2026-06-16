---
slug: new-cadair
repo: https://github.com/CKS-Equium/new-cadair
status: active
phase: build
created: 2026-06-16
team: [orchestrator, software-architect, senior-software-engineer, quality-engineer, reviewer-critic]
board: n/a
---

## Summary

**A/B experiment, not a product.** The PM team's independent build of **Cadair Phase A (Foundation)**
— a software bioprocess control engine (Rust/PyO3) + Modbus adapter + ZeroMQ bus + process simulator
+ HMI skeleton — built from the *same specs* as the reference build ("OG-cadair", `D:/github/cadair`),
so the two can be **run side-by-side and compared**. The goal is to **improve the PM team**, not to
ship Cadair. Built by the improved team (the three validation-grade process changes were banked first).

## Current state

**Scaffolded; Phase A build starting.** `new-cadair` holds OG's controlled specs (FS, architecture,
tech-stack, testing-strategy, roadmap, HMI design system, TFF reference) in `docs/spec/` as the input.
**A/B fairness:** OG's *phase design* doc was deliberately withheld — the team's architect designs
Phase A itself. Same stack (Rust+PyO3+Python+ZMQ+FastAPI+React). Bars to match OG: scan benchmark,
system-validation-gate, FS traceability, per-gate evidence record. Built sub-phase by sub-phase
(branch+PR each, self-merged on green DoD; no human-playtest gate — Phase A is machine-verifiable).

## Reference (the "A")

OG-cadair (`D:/github/cadair`) — Phase A complete: ~235 tests, ~4.7µs scan, system-validation-gate,
FS traceability, validation-grade evidence docs. Confirmed running on this machine (engine imports,
14/14 integration tests, HMI prebuilt). The head-to-head comparison + distilled PM-team improvements
land after new-cadair's Phase A completes.

## Decision log

- 2026-06-16 — A/B experiment set up to improve the PM team. Three improvements banked first
  (per-gate evidence record, requirement→test→source traceability, failure-mode review lens). Scope =
  Full Phase A. OG phase-design withheld for fairness; team designs its own.
