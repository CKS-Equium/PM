---
slug: localcoder
repo: https://github.com/CKS-Equium/localcoder
status: active
phase: design
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

**Gate 1 PASSED (PRD approved 2026-06-18) — entering Design.** Repo created + seeded (PRD + scaffold).
Backend settled with **live probes on the operator's RTX 5090**: `qwen3-coder:30b` @ `num_ctx 65536`
runs 100% on GPU (~29 GB VRAM, ~271 tok/s) with clean native tool-calling + correct loop
continuation; `nemotron-3-nano` is the fast toggle; OpenAI-API shape keeps the model a one-line swap.
**Build from scratch** (not forking Continue/Cline) for transparency. **Light ~1-week timebox**
(target ~2026-06-25). **New stack for the team:** TypeScript + VS Code Extension API. Next gate:
**gate 2 (architecture approval)**. Seed issues: architecture+ADRs, planning.

## Decision log

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
