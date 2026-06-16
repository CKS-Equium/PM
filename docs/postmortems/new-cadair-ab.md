# A/B Experiment Report — new-cadair vs OG-cadair (Cadair Phase A)

**Date:** 2026-06-16
**Type:** Controlled A/B build — *the experiment's product is the PM team, not Cadair.*
**Slice:** Full Phase A (Foundation). **Verdict:** **Both build, both run live.** B (PM team) reached
feature + run parity with A (reference) from the same specs, and the PM-team process changes paid out.

---

## 1. The experiment

- **A — OG-cadair** (`D:/github/cadair`): the reference build by a different agent team. Phase A
  (+ a Phase B/B+ UFDF) complete; ~235 tests; ~4.7 µs engine scan; system-validation gate; FS
  traceability; validation-grade evidence docs.
- **B — new-cadair** (`D:/github/new-cadair`): the PM team's independent build of **Phase A only**,
  from the **same controlled specs** (FS, architecture, tech-stack, testing-strategy, roadmap, HMI
  design, TFF reference) — *built fresh, OG's implementation + phase-design doc deliberately withheld
  so the team designed Phase A itself.* Same stack (Rust+PyO3 / Python / ZeroMQ / FastAPI / React).
- **Why:** to find which of the PM team's improvements actually move the needle, by running OG and
  new-cadair side-by-side. The three banked-first improvements (per-gate **evidence record**,
  **requirement→test→source traceability**, **failure-mode review lens**) were applied *before* the
  build so B was produced by the improved team.

---

## 2. Head-to-head — both built, both run

Each stack was booted from its own `run-demo.sh` path (sim + Modbus adapter + Rust engine + ZeroMQ
bus + orchestrator scan loop + WebSocket bridge + FastAPI serving the React HMI), then a WebSocket
client logged in and captured live `scan_complete` frames over a fixed window, then each was shut
down cleanly (no leaked listeners).

| | **A — OG-cadair** (`:8000`) | **B — new-cadair** (`:8001`) |
|---|---|---|
| Boots full stack & serves HMI | ✅ | ✅ |
| Live WS frame rate | ~9.2 Hz (100 ms scan) | ~10 Hz (100 ms scan) |
| Scans observed | 2634 → 2726 (running batch) | 1929 → 1990 (running) |
| Live closed-loop | ✅ inputs/outputs moving through recipe (pumps 50/60 %, valves modulating) | ✅ TMP 0.346 → 0.278 bar, pump PID 28.3 % → 26.8 % |
| Recipe start | **Operator must press "Start Batch"** (`POST /recipe/start`) | **Auto-runs** the TIMED phase driver on boot |
| UFDF fidelity | 10 AI / 13 AO, **9-phase** full TFF sequence (Phase B+ depth) | 10 AI / pump+valve AO, **TIMED** phase driver (Phase-A skeleton — by scope) |
| Engine scan (bench, comparable) | ~4.7 µs median | **~0.94 µs median, zero-alloc 0/10k** |
| Tests | ~235 (incl. Phase B/B+) | 158 pytest + 73 cargo = **231 for Phase A alone** |
| FS traceability | matrix, 80 Phase-A reqs | consolidated RTM, **104 FS-IDs (92 traced / 12 named-deferred)** |
| One-command CI | `ci.sh`, 8 stages | `ci.sh`, **7 stages, proven deterministic 5×** |
| Evidence record | Phase-A + Phase-B+ evidence docs | per-sub-phase evidence + finalized Phase-A record |

Both satisfy all six roadmap §4 Phase-A exit criteria with evidence. **B matched A's
build-and-run bar** in a single autonomous run (six sub-phases A.1–A.6, branch+PR each,
self-merged on green DoD), from spec, without copying A.

> Scope note: A carries more UFDF *fidelity* (a full 9-phase recipe from its Phase B/B+ work); B
> built **Phase A only** as specified. This is a scope difference, not a quality gap — B's
> architecture-proving foundation is complete and the simulator runs the closed loop.

---

## 3. The central finding (already banked, now *proven*)

The **failure-mode review lens** (BAD/NaN/Inf/div0/boundaries/latched/dependency-death/unbounded-wait)
caught **5 real defects that green test suites missed**, pre-merge, on the Tier-1 / data-integrity
sub-phases:

1. **A.1 B1** — latched interlock re-fired a *fresh* trip/alarm/audit event every scan it stayed
   tripped (alarm + GxP-audit flooding; wrong IDs on multi-trips).
2. **A.1 M1** — an oversized config aborted the host process (`panic=abort`) instead of a clean
   `ValueError`.
3. **A.1 M2** — PID HOLD fired on UNCERTAIN quality, deviating from the FS (HOLD on BAD only) and
   contradicting the totaliser's own rule.
4. **A.2 BLOCKER** — an unguarded event-sink let a raising callback silently kill the reconnect task
   *and* escape into the scan loop (violating graceful-degradation FS-PROT-002).
5. **A.3 MAJOR** — `publish()` released its lock before persist+send → non-thread-safe ZMQ socket
   use + durable-persist order diverging from sequence order (a latent compliance-gap false-flag).

Every one is a correctness/safety bug a passing green build would have shipped. **This is the
strongest single signal of the experiment**: the lens earns its place as a standing gate for
Tier-1 / data-integrity / compliance code.

---

## 4. Non-obvious lessons surfaced by *running* it (the new payoff)

These emerged only from building + running the A/B, beyond the three banked improvements:

### 4.1 The one-command CI + evidence record is a **resilience / handoff** asset, not just quality
During A.6 the building agent died **twice on transient API errors** (500, then 529) with no final
report. The work survived and shipped anyway because each sub-phase left **(a)** an incremental
commit on its own branch, **(b)** a self-describing **evidence record**, and **(c)** a **one-command
`ci.sh`**. The orchestrator (me) re-verified the half-built A.6 by *running `ci.sh` directly* and
*reading the evidence doc for honesty* — then integrated it — **without resurrecting the dead agent's
context.** A self-contained verifier + an honest evidence record make a build **interruption-tolerant**:
any actor (a fresh agent, the orchestrator, the human) can independently verify and integrate.
→ *Improvement: treat "one-command verifier + self-describing evidence record per sub-phase" as a
required handoff/resilience artifact, explicitly — not only a CI nicety.*

### 4.2 Metric-span honesty — a green number can measure the wrong span
B's live `scan_time_ms` read **0.7 µs**; A's read **0.73 ms** — a 1000× apparent gap that is an
**artifact of what each measures**. B times only the Rust `engine.scan()`; A times the **whole
orchestrator cycle** (adapter read + engine + publish). A's number is the operationally meaningful
one; B's flatters itself by hiding adapter + publish latency. Both are "green," but only one answers
"can we hold the 100 ms loop end-to-end?"
→ *Improvement: when a metric is a budget/gate, the evidence/RTM must declare **what span it covers**,
and prefer the **end-to-end** span. Add a "metric honesty" sibling to the failure-mode lens.*

### 4.3 Run-it parity gap — the validation gate proved the data path, not the operator path
B **auto-runs** the recipe on boot (excellent for autonomous, human-free verification — and the
reason B could complete overnight). But the consequence is that B's runnable demo + validation gate
never exercise the **operator-initiated** recipe-start path that A's spec implies a real operator
uses (A requires a "Start Batch" press; we had to `POST /recipe/start` to make A run). The data path
is proven; the **primary operator entry path is not.**
→ *Improvement: the system-validation gate / acceptance must exercise the **primary operator-initiated
path**, not only an auto-driven one — or explicitly register the gap. (Echoes the ColonyGame
pre-playtest interaction-review lesson: verify the path the human actually takes.)*

---

## 5. What changes in the control plane

| # | Lesson | Where it lands | Channel |
|---|--------|----------------|---------|
| Banked | Failure-mode review lens (now proven: 5 defects) | reviewer-critic / quality-engineer persona + gates.md (already applied) | reviewed (done) |
| Banked | Per-gate evidence record + traceability matrix | templates + personas + gates.md (already applied) | reviewed (done) |
| 4.1 | Evidence record + one-command verifier = required resilience/handoff artifact | process-engineer notes + orchestrator notes | notes (free) |
| 4.2 | Metric-span honesty (declare span; prefer end-to-end; lens sibling) | quality-engineer notes + reviewer-critic notes | notes (free) |
| 4.3 | Validation gate must exercise the operator-initiated path | quality-engineer notes + product-manager notes | notes (free) |

Lessons 4.1–4.3 are appended to the relevant agents' `notes.md` (free self-edit per DESIGN §5). If any
hardens into a contract rule, it graduates to the persona via a reviewed PR — not assumed here.

---

## 6. Bottom line

The PM team, running the improved process, **independently rebuilt Cadair Phase A from spec to a
build-and-run parity with the reference** — and the headline process bet (the failure-mode lens)
**caught 5 real, shippable defects that the green suite missed.** Running the two side-by-side
surfaced three further, non-obvious improvements that pure inspection would not have (metric-span
honesty, operator-path validation, evidence-as-resilience). The experiment did its job: it improved
the team.
