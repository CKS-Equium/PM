# Gate Evidence Record: gate <n> (<phase transition>)

> Owner: **Quality Engineer** (produces at each gate; Orchestrator confirms at the transition).
> Operationalises the verification-vs-PQ principle in [gates.md](../gates.md): verification is
> necessary but not sufficient, so each gate leaves a short, honest proof trail.

## Exit-criterion evidence
Each DoD exit criterion → its status → the proof (verifying command, test ref, or output).

| Exit criterion | Status | Evidence (command / test ref / output) |
|----------------|--------|-----------------------------------------|
| … | ✅ met / ⚠ partial / ❌ not met | `<cmd>` / `tests/…::case` / log line |

## Known follow-ups register
Honest list of gaps, deferrals, and known-imperfect areas — named, not hidden. A gate may pass
with open follow-ups, but they must be recorded here (and routed: ticket / notes / next gate).

| # | Follow-up / gap | Severity | Routed to |
|---|------------------|----------|-----------|
| 1 | … | low / med / high | #issue / deferred to gate <n> / notes |
