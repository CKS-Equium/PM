---
slug: localcoder
repo: https://github.com/CKS-Equium/localcoder
status: active
phase: build
created: 2026-06-18
team: [orchestrator, product-manager, software-architect, senior-software-engineer, quality-engineer, reviewer-critic]
board: n/a (token lacks project scope; using labels + milestones + issues)
---

## Summary

A **single-user VS Code extension** delivering a **transparent, Claude-Code-style agentic coding
loop driven by a local LLM via Ollama**. The operator wants to *own and see "the middle"* (the agent
harness) that closed tools like Cline hide, and to explore what's possible with local models —
**learning, not efficiency** ("less performative than Claude Code" is a pass). Three layers: a
right-docked webview chat panel + an agent loop (Ollama OpenAI-compatible `/v1`, native tool-calling)
+ five tools (`read_file`, `write_file`, `list_dir`, `search`, `run_command`) behind a
**Claude-Code-style permission layer** (read-only auto-approved; mutating tools gated by
Approve/Deny/Always-allow; allowlist persisted; deny handled gracefully).

## Current state

**BUILD + REVIEW COMPLETE — only the live-GPU verification remains (2026-06-18).** All 8 build
tickets (#3→#10) merged; **119 mocked tests green**, typecheck + lint clean, zero GPU used (Ollama
client mocked throughout, per the operator's GPU constraint). Reviewer-critic **APPROVE WITH NITS**;
security-engineer **PASS WITH FINDINGS** — findings fixed (F1 crypto nonce, F2 shell-operator
approval warning + risk R16, autoAcceptEdits logged-setter, missing-id correlation, VRAM re-warn),
merged PR #20. The permission gate is verified as a real enforced trust boundary (not just UI).
**Pending = the live ACs (AC1/AC3/AC11/AC13) — require the operator's GPU**: a backend live smoke
(`node spike/agent-loop.mjs`) and the interactive end-to-end (F5 → panel → prompt → read→edit→run,
all gated) per `docs/DEMO.md`. This is the human/live verification gate (operator-initiated, like a
playtest). Gate 6 (release) after the live run passes.

**(historical) Gate 1 PASSED (PRD approved 2026-06-18) — entering Design.** Repo created + seeded (PRD + scaffold).
Backend settled with **live probes on the operator's RTX 5090**: `qwen3-coder:30b` @ `num_ctx 65536`
runs 100% on GPU (~29 GB VRAM, ~271 tok/s) with clean native tool-calling + correct loop
continuation; `nemotron-3-nano` is the fast toggle; OpenAI-API shape keeps the model a one-line swap.
**Build from scratch** (not forking Continue/Cline) for transparency. **Light ~1-week timebox**
(target ~2026-06-25). **New stack for the team:** TypeScript + VS Code Extension API. Next gate:
**gate 2 (architecture approval)**. Seed issues: architecture+ADRs, planning.

## Decision log

- 2026-06-18 — **Gate 2 approved (architecture + plan).** Spike GO: in-code agent loop drove
  list_dir→read_file→final on qwen3-coder via `/v1` (5.6s cold/1.1s warm, 100% GPU at 64k). ADRs
  0001–0004 accepted. **Allowlist granularity:** write_file = per-path-prefix, run_command =
  per-exact-command. **VRAM posture:** warn-not-cap (detect CPU spill from `/api/ps`, non-blocking).
  Plan = 8 tickets #3→#10 (critical path to the AC11 end-to-end milestone); R15/streaming cut-first.
- 2026-06-18 — **⚠️ BUILD CONSTRAINT (operator):** the GPU is in use elsewhere — **no live LLM/GPU
  inference during the build.** All automated tests mock the Ollama client; any test needing the GPU
  (AC3 live tool-calling, AC11 end-to-end, AC13 model swap) is **gated behind an explicit operator
  ask** before running.
- 2026-06-18 — **Gate 1 approved.** Name `localcoder`. Build from scratch (transparency/learning over
  forking Continue/Cline). Full tool set incl. `run_command`, gated by a Claude-Code-style permission
  layer. `run_command` guardrail = **approval-only** for v1 (single trusted local user).
- 2026-06-18 — **Backend locked (evidence-based):** Ollama OpenAI-compatible `/v1`; default
  `qwen3-coder:30b` @ 64k ctx (MoE, ~3B active → fast *and* capable); `nemotron-3-nano` fast toggle.
  Big `num_ctx` is mandatory (small default context is the #1 local-agent failure mode).
- 2026-06-18 — **Light ~1-week timebox** for scope discipline; goal is learning/"what's possible,"
  not efficiency. Out of scope v1: multi-user, cloud-first, marketplace polish, MCP, large
  autonomous refactors, tools beyond the five.

## Gate waivers

- (none)
