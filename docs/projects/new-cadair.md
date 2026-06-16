---
slug: new-cadair
repo: https://github.com/CKS-Equium/new-cadair
status: complete
phase: phase-a-complete
created: 2026-06-16
team: [orchestrator, software-architect, senior-software-engineer, quality-engineer, reviewer-critic]
board: n/a
report: ../postmortems/new-cadair-ab.md
---

## Summary

**A/B experiment, not a product.** The PM team's independent build of **Cadair Phase A (Foundation)**
— a software bioprocess control engine (Rust/PyO3) + Modbus adapter + ZeroMQ bus + process simulator
+ HMI skeleton — built from the *same specs* as the reference build ("OG-cadair", `D:/github/cadair`),
so the two can be **run side-by-side and compared**. The goal is to **improve the PM team**, not to
ship Cadair. Built by the improved team (the three validation-grade process changes were banked first).

## Current state

**Phase A COMPLETE — experiment concluded; both builds run.** Built sub-phase by sub-phase (A.1–A.6,
PRs #1–#6, branch+PR each, self-merged on green DoD). `ci.sh` runs 7 staged gates green, 158 pytest +
73 cargo tests, deterministic 5×, system-validation gate boots the real stack (no mocks), consolidated
RTM (104 FS-IDs, 92 traced / 12 named-deferred); all six roadmap §4 Phase-A exit criteria met with
evidence. **Both stacks were run side-by-side and compared** (each boots sim+adapter+engine+bus+
orchestrator+WS+FastAPI and serves a live HMI): OG ~9.2 Hz, new-cadair ~10 Hz, both live closed-loop.
Full head-to-head + distilled PM-team improvements in **[the A/B report](../postmortems/new-cadair-ab.md)**.

**Headline outcome:** the **failure-mode review lens caught 5 real defects** the green suites missed
(2 in A.1, 1 in A.2, 1 in A.3 — interlock-trip flooding, host-abort-on-bad-config, HOLD-on-UNCERTAIN
deviation, unguarded-event-sink escape, publish lock-scope thread-safety). Running the two side-by-side
surfaced 3 further non-obvious improvements (evidence-as-resilience, metric-span honesty,
operator-path validation) — applied to agent `notes.md` per DESIGN §5.

**A/B fairness held:** OG's *phase design* doc was withheld; the team's architect designed Phase A
from the FS + architecture. Same stack (Rust+PyO3+Python+ZMQ+FastAPI+React).

## Reference (the "A")

OG-cadair (`D:/github/cadair`) — Phase A complete: ~235 tests, ~4.7µs scan, system-validation-gate,
FS traceability, validation-grade evidence docs. Confirmed running on this machine (engine imports,
14/14 integration tests, HMI prebuilt). The head-to-head comparison + distilled PM-team improvements
land after new-cadair's Phase A completes.

## Decision log

- 2026-06-16 — A/B experiment set up to improve the PM team. Three improvements banked first
  (per-gate evidence record, requirement→test→source traceability, failure-mode review lens). Scope =
  Full Phase A. OG phase-design withheld for fairness; team designs its own.
