# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repository is the **control plane for an autonomous, multi-agent software-project-management
team** — not application code, and not any single project. It defines a team of 15 role-based
agents (with strict, non-overlapping scopes) that can take a software project from a plain-English
intent through discovery, design, build, test, review, release, and a self-improvement
post-mortem. The "product" is the team: the persona contracts, skills, artifact templates, and the
accumulating learning.

**Read [docs/DESIGN.md](docs/DESIGN.md) first — it is the single source of truth.** This file is
an orientation; DESIGN.md is authoritative, with [docs/ORG.md](docs/ORG.md) (org chart) and
[docs/gates.md](docs/gates.md) (lifecycle gates) as the other primary references.

### The core idea: control plane vs data plane

- **Control plane = this repo.** The reusable team + everything it has learned.
- **Data plane = a separate GitHub repo per project**, created on demand with `gh repo create`,
  living *outside* this repo's tree. This repo tracks only a metadata **registry** entry per
  project under `docs/projects/`. Never nest a project's git repo inside this one.

Durable lessons flow back into the control plane (agents' `notes.md`); project-specific context
stays in the project repo.

## Repository layout

```
.claude/
  agents/<role>/
    persona.md     # CONTRACT — mission, scope, does-NOT. Stable, REVIEW-GATED.
    notes.md       # PLAYBOOK — heuristics the agent self-appends FREELY.
  skills/          # workflow skills, incl. the existing git lifecycle skills
  settings.json    # permissions allowlist
docs/
  DESIGN.md        # authoritative blueprint (read first)
  ORG.md           # org chart + reporting principles
  gates.md         # phase gates, Definition of Done, human approval points
  templates/       # artifact templates (PRD, ADR, ticket, test-plan, UX/UI spec)
  projects/        # project registry (one <slug>.md per project + index)
  postmortems/     # one 360-review report per completed project
```

## Working conventions

- **Scope is enforced by artifacts + handoffs.** Each agent reads named inputs and writes named
  outputs, and must not edit artifacts it doesn't own. A role's authoritative scope is its
  `persona.md` (all 15 exist under `.claude/agents/<role>/`); DESIGN.md §3 is the summary view.
- **Two-file agents.** `notes.md` is freely self-edited; `persona.md` (the contract) changes
  **only via a reviewed PR** — convention + Reviewer/human approval, no enforcement hook.
- **Model tiering** (DESIGN.md §8): Opus for the six senior/judgment roles, Sonnet for the mid
  tier, Haiku for the Junior Engineer. Set per agent via `model:` frontmatter.
- **Human-in-the-loop** at exactly three gates: PRD approval, architecture approval, release
  (gates.md). Autonomy runs free between them.
- **Templates are the contracts between roles** — when changing what an agent produces/consumes,
  update the matching template in `docs/templates/` so the handoff stays consistent.
- **Cross-document consistency:** the roster, scopes, gates, model tiering, and lifecycle appear
  in DESIGN.md, ORG.md, gates.md, and the per-role personas. A change in one usually needs
  mirroring in the others.

## The git lifecycle skills

`.claude/skills/` contains seven chained git-workflow skills — `smart-status`, `smart-branch`,
`smart-save`, `smart-commit`, `smart-pull-request`, `smart-merge`, `smart-cleanup`. Within the
team these are the **DevOps / Release Engineer's** toolkit; standalone, they teach Claude
conventional git operations. Authoring conventions for them:

- **Frontmatter is mandatory** (`name` kebab-case matching the directory, `description` stating
  *what* + *when to use*).
- **Body:** Title → summary → `## Workflow` (numbered Phases) → `## Rules` (`Never:`/`Always:`/
  `Prefer:`) → `## Edge Cases` → `## Invocation`.
- **Explicit staging** — never `git add .`/`-A` (the sole exception is `smart-save`'s documented
  WIP checkpoint).
- **Conventional commits/branches**: types `feat|fix|docs|refactor|perf|test|chore|ci|style`;
  branches `<type>/<kebab-desc>`; subjects imperative, lowercase, ≤72 chars, no trailing period.
- **Protected branches** (`main`/`master`/`develop`) are never committed to or deleted directly;
  destructive git operations require explicit confirmation.

## Tooling dependencies (runtime)

`git`; `gh` (GitHub CLI, authenticated) for project-repo creation, issues, GitHub Projects, and
PRs; Claude Code as the runtime. Per-project quality-gate tools (linters, type-checkers, test
runners) are supplied by each project, not installed here.
