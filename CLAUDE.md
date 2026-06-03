# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repository is a collection of **Claude Code skills** — not application code. Each skill
is a Markdown workflow definition that teaches Claude an intelligent, conventional git
operation. The "product" is the prose in each `SKILL.md`; there is nothing to build, compile,
or run. Skills are consumed by Claude Code at runtime, which reads the frontmatter to decide
when a skill applies and follows the body as a procedure.

The seven skills form a cohesive git lifecycle that reference and chain into one another:

```
smart-status ──▶ smart-branch ──▶ (work) ──▶ smart-save ──▶ smart-commit ──▶ smart-pull-request ──▶ smart-merge ──▶ smart-cleanup
   (hub)            (start)                   (checkpoint)    (organize)        (PR + CI)              (integrate)      (tidy up)
```

- **smart-status** — workflow hub: repo overview, branch navigation, suggests the next skill.
- **smart-branch** — create a branch with conventional `<type>/<desc>` naming from a plain-English intent.
- **smart-save** — WIP checkpoint commit (`wip:` prefix, `git add -A`) pushed to remote for backup.
- **smart-commit** — reorganize working-tree changes into atomic, conventional commits by concern.
- **smart-pull-request** — quality gates (lint/types/tests/build) → push → generate PR body → `gh pr create` → CI watch.
- **smart-merge** — branch detection, sync, merge/rebase/squash strategies, conflict-resolution guidance.
- **smart-cleanup** — categorize and delete merged/stale/gone branches, local and remote.

## Repository layout

```
.claude/
  settings.json          # permissions allowlist for this project
  skills/<name>/SKILL.md  # one directory per skill; the SKILL.md IS the skill
```

## SKILL.md authoring conventions

When adding or editing a skill, match the established structure — the existing files are the
spec. Key conventions enforced across the set:

- **Frontmatter is mandatory.** Every `SKILL.md` must start with a YAML block containing
  `name` (kebab-case, matching the directory) and `description`. The `description` is how
  Claude decides relevance, so it must state *what* the skill does **and** *when to use it*
  (e.g. "Use when starting new work, creating a branch...").
- **Body structure:** Title → one-line summary → `## Workflow` broken into numbered Phases →
  `## Rules` with `Never:` / `Always:` / `Prefer:` lists → `## Edge Cases` → `## Invocation`
  (the natural-language phrases that trigger the skill).
- **Explicit staging.** Skills must never use `git add .` / `git add -A` — stage files by name.
  The one deliberate exception is `smart-save`, which uses `git add -A` for throwaway WIP
  checkpoints and documents *why* inline.
- **Conventional commits / branches** throughout: types are `feat`, `fix`, `docs`, `refactor`,
  `perf`, `test`, `chore`, `ci`, `style`. Branches are `<type>/<kebab-desc>`; commit subjects
  are imperative, lowercase, ≤72 chars, no trailing period.
- **Protected branches** (`main`/`master`/`develop`) are never committed to directly or
  deleted; destructive git operations require explicit user confirmation.
- **Optional per-skill config:** several skills look for a dotfile (`.smartbranch.json`,
  `.smartmerge.json`, `.smartcleanup.json`, etc.). These are conventions described in the
  skills, not code that exists here.

## Cross-skill consistency

Because the skills chain together, a change in one often needs mirroring in others. Notably:
the conventional type/branch vocabulary, the protected-branch list, the "suggest next skill"
hand-offs (e.g. smart-status → smart-save/smart-commit, smart-merge → smart-cleanup), and the
shared assumption that `gh` CLI is available for PR-related steps.

## Tooling dependencies (runtime, not this repo)

The workflows assume a developer environment with `git`, and `gh` (GitHub CLI, authenticated)
for `smart-pull-request` and the PR view in `smart-status`. Quality-gate commands in the
skills are written for both Python (`ruff`, `mypy`/`pyright`, `pytest`) and Node/TS (`npm run
lint`, `tsc`, `npm test`) — they are templates the target project supplies, not tools this
repo installs.
