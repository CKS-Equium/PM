---
slug: event-intel
repo: https://github.com/CKS-Equium/event-intel
status: active
phase: release
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

**Gate 2 PASSED (architecture approved 2026-06-17) — in Build.** Day-1 spike on the real 5090 box
returned **GO**: faster-whisper on CTranslate2 `cp314` (runs on Python 3.14, **no torch**, 14.6×
realtime); vision via Ollama **`qwen3-vl:30b` already on disk** (zero download, GPU); ffmpeg
scene-detect ≈ 1:1 slides; sample dossier had all 5 fields + read figures off slides the audio never
said; **full 13-min session ≈ 4.7 min end-to-end** (target 1–2h / window 4–6h). The vision-required
deadline risk is **retired with evidence**. Accuracy is **networking-grade** (slides shipped
alongside) — operator-acknowledged. `sessions/` KB **auto-committed** (operator-approved). Build =
harden the proven spike into the production pipeline + the `/remote-control` runbook + tests; branch+
PR per unit, self-merge on green DoD. Next gate: build DoD → gate 4/5 (test+review) → release.
Operator pre-Monday checklist: Whisper large-v3 warm-up (~1.5 GB), sleep off, wired ethernet,
Tailscale+RDP backup.

**BUILD COMPLETE — ready for Monday (2026-06-17).** Pipeline built, reviewed (APPROVE w/ nits),
hardened (file-quiescence + transcript-coverage guard against silent-truncation + robust label
match), merged (PR #4, main `32ef786`). **67 tests green; real e2e on the NIIMBL sample 170s,
transcript coverage 0.993, all 5 dossier fields.** Whisper large-v3 already cached (warm-up done
during build). Remaining = operator-only: set `EVENT_INTEL_RECORDINGS_DIR` to the real OBS folder,
liveness (sleep off / ethernet / Tailscale+RDP), start a `/remote-control` session before leaving.
**Post-mortem after the event** (once used live Monday) — close the loop then.

**LIVE INCIDENT + FIX (2026-06-23/24) — first real full-length session.** Processing the operator's
first true 82-min session (vs the 170s build sample) exposed that the build's smoke test never
exercised the long-context fusion path. Fusion broke four scale-dependent ways: (A) the fixed 16K
`num_ctx` silently truncated the ~28K-token prompt → empty dossier; (B) a wider window made
generation loop (185KB, every speaker ~10×); (C) **root cause** — `qwen3-vl` is an always-on
reasoning model whose unbounded, unsuppressable `<think>` block (ollama `think:false` **and**
`/no_think` both ignored) returned an **empty** response on ~1/3 of runs; (D) coverage variance.
**Fix (branch `fix/fusion-long-session-hardening`, commit `ff67b4e`):** fusion is pure-text and never
needed vision → moved to a **non-thinking text model** (`qwen3-coder:30b`, new `EVENT_INTEL_FUSION_MODEL`);
+ input-sized `num_ctx` with fail-loud ceiling, `num_predict` bound, deterministic speaker-block
dedupe, and a "cover EVERY speaker" prompt. Tests 67→77; validated on the real session (0 empties,
26–38 speakers). Residual: single-shot coverage still varies (re-run is safe/idempotent) — **two-pass
roster-then-detail** is the proposed follow-up. Learnings recorded to SSE + QE `notes.md`.

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
