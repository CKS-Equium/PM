# PM — Autonomous Multi-Agent Project Management Team

An autonomous software-project-management **team**, built as a set of role-based
[Claude Code](https://claude.com/claude-code) agents with strict, non-overlapping scopes. Describe
what you want to build, and the team runs it: discovery → design → build → test → review →
release → post-mortem — generating the plan, requirements, architecture, design, code, tests, and
docs along the way, and getting measurably better with every project.

> **This repo is the team, not a project.** Each project the team executes lives in its own
> separate GitHub repo. → Read **[docs/DESIGN.md](docs/DESIGN.md)** for the full blueprint.

## How it works

- **Control plane / data plane.** This repo is the reusable team (personas, skills, templates,
  accumulated learning). Each project gets its **own GitHub repo**, created on demand; this repo
  keeps only a registry entry linking to it.
- **15 roles, hierarchical delegation.** Human → Orchestrator → leads → workers. See the
  [org chart](docs/ORG.md). Quality, review, and security report independently; a Process Engineer
  continuously improves the team.
- **Strict scope via artifacts.** Each agent reads named inputs and writes named outputs
  ([templates](docs/templates/)), and can't touch what it doesn't own.
- **GitHub-native state.** Tickets are Issues, the board is a GitHub Project, phases are
  Milestones, handoffs are comments, deliveries are PRs.
- **Gated, with humans at the big calls.** Explicit [gates](docs/gates.md) between phases; a human
  signs off at PRD, architecture, and release only.
- **Recursive self-improvement.** Agents keep a `notes.md` playbook; a post-mortem (self-review +
  360 review) after every project feeds improvements back into the team — audited in git.

## Repository layout

```
.claude/
  agents/<role>/   persona.md (contract) + notes.md (playbook)
  skills/          workflow skills, incl. the git lifecycle skills
docs/
  DESIGN.md        authoritative blueprint (start here)
  ORG.md           org chart
  gates.md         phase gates, Definition of Done, human approval points
  templates/       PRD · ADR · ticket · test-plan · UX/UI spec
  projects/        project registry (one entry per project)
  postmortems/     360-review reports
```

## Status

Rev 1 — architecture, scaffolding, all **15 persona contracts** (`.claude/agents/<role>/`, each
`persona.md` + `notes.md`), and the **`start-project`** kickoff skill are in place. Next: the
lifecycle/gate orchestration that drives the team through the phases, then the first dogfood
project (a status dashboard). See [DESIGN.md §9](docs/DESIGN.md) for the build order.

## Requirements

- **Claude Code** (the runtime the agents run in).
- **git**.
- **[GitHub CLI](https://cli.github.com/) (`gh`)**, authenticated — for project-repo creation,
  Issues, GitHub Projects, and PRs.

Per-project tooling (linters, test runners, etc.) is supplied by each project, not this repo.

## The git lifecycle skills

`.claude/skills/` holds seven chained git-workflow skills — `smart-status`, `smart-branch`,
`smart-save`, `smart-commit`, `smart-pull-request`, `smart-merge`, `smart-cleanup` — which form
the DevOps / Release Engineer's toolkit and are usable standalone in any repo.
