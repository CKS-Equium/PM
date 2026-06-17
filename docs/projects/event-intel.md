---
slug: event-intel
repo: https://github.com/CKS-Equium/event-intel
status: active
phase: design
created: 2026-06-17
team: [orchestrator, product-manager, software-architect, senior-software-engineer, quality-engineer, reviewer-critic]
board: n/a (token lacks project scope; using labels + milestones + issues)
---

## Summary

A **local-only** pipeline that turns **recorded event-session videos** (captured by OBS on the
operator's home RTX 5090 box) into per-session **"who-to-focus-on" intel dossiers** — local
transcript + extracted slide/visual images + a **local vision-LLM** distillation (speaker · org ·
what they presented · notable claims/data · questions raised · **engagement angle for the evening**).
Triggered from the operator's phone via Claude Code **`/remote-control`**; all inference runs locally
(privacy is a driver). Purpose: walk into the evening networking event knowing **who to focus on and
why**, from the day's opening sessions. **Hard external deadline: Monday 2026-06-22.**

## Current state

**Gate 1 PASSED (PRD approved 2026-06-17) — entering Design.** Repo created + seeded (PRD +
scaffold). Next gate: **gate 2 (architecture approval)**. Seed issues: architecture+ADRs, planning.
Build cadence: branch+PR per unit, self-merge on green DoD (no human-playtest gate — machine-
verifiable, like new-cadair/colonygame). **Vision-LLM dossier is a v1 must-have (operator's call,
higher deadline risk) → architect/engineer must prototype the vision dossier + smoke-test the
Windows/CUDA local-model toolchain on day 1** to front-load the riskiest integration.

## Decision log

- 2026-06-17 — **Gate 1 approved.** Name `event-intel`; control surface = Claude Code
  `/remote-control` (no Telegram — `/remote-control` is what the operator already uses, and the
  Telegram bot would not be E2E). Build = the pipeline + repo conventions, not the control surface.
- 2026-06-17 — **Vision-LLM dossier is a v1 must-have**, not a text-only floor (operator chose the
  richer/higher-risk option). No text-only fallback. De-risked by a mandated day-1 vision prototype.
- 2026-06-17 — **Local models only** (Whisper + a vision LLM on the 5090; architect finalizes) —
  privacy driver; rules out hosted APIs.
- 2026-06-17 — **Signal photo intake = gated STRETCH** (signal-cli linked as a secondary device,
  saves photos to `intake/`), chosen over Telegram for E2E privacy; never on the Monday-critical path.
- 2026-06-17 — **Session-liveness operational risk ACCEPTED**: `/remote-control` needs the box +
  session alive; mitigations = disable sleep, wired ethernet, Tailscale/RDP backup to restart from
  the phone.
- 2026-06-17 — Out of scope v1: live/real-time, OBS control, the flaky `/remote-control` photo-
  attach path, web UI, multi-user, cloud fallback.

## Gate waivers

- (none)
