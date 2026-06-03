# Smart Git Skills for Claude Code

A collection of [Claude Code](https://claude.com/claude-code) skills that turn an
end-to-end git workflow into intelligent, conventional commands. Describe what you want in
plain English ("start working on a YAML parser", "save my work", "create a PR") and the
matching skill runs the right git/`gh` commands with safety checks, conventional naming, and
quality gates.

These are **skill definitions**, not an application — there is nothing to build or run. Claude
Code loads each `SKILL.md` and follows it as a procedure.

## The skills

| Skill | Purpose |
|-------|---------|
| **smart-status** | Workflow hub — repo overview, branch navigation, suggests the next action. |
| **smart-branch** | Create a branch with conventional `<type>/<description>` naming from your intent. |
| **smart-save** | Quick WIP checkpoint commit pushed to remote (end-of-day / context-switch backup). |
| **smart-commit** | Reorganize changes into atomic, conventional commits grouped by concern. |
| **smart-pull-request** | Quality gates → push → auto-generate PR description → `gh pr create` → watch CI. |
| **smart-merge** | Branch detection, sync, merge/rebase/squash strategies, conflict guidance. |
| **smart-cleanup** | Find and delete merged, stale, and orphaned branches (local + remote). |

They chain into a lifecycle:

```
smart-status → smart-branch → …work… → smart-save → smart-commit → smart-pull-request → smart-merge → smart-cleanup
```

## Usage

With Claude Code running in your project, invoke a skill by describing the task or naming it:

- "smart status" / "what's going on" / "show branches"
- "start new work on adding YAML support"
- "save my work — end of day"
- "smart commit" / "organize my commits"
- "create a PR" / "smart pr"
- "smart merge target=develop strategy=squash"
- "clean up merged branches" / "smart cleanup --dry-run"

Each skill's `## Invocation` section lists the phrases that trigger it, and `## Workflow`
documents exactly what it does.

## Requirements

- **Claude Code** (the skills run inside it).
- **git**.
- **[GitHub CLI](https://cli.github.com/) (`gh`)**, authenticated (`gh auth login`) — required
  by `smart-pull-request`; optional for `smart-status` (used to list PRs).

The quality-gate steps in `smart-pull-request` and `smart-merge` invoke your project's own
linters/type-checkers/test runners (e.g. `ruff`/`pytest` or `npm run lint`/`npm test`); they
are templates, not bundled tools.

## Installation

Copy (or symlink) the skill directories into the `.claude/skills/` of any project, or keep
them in a project-level / user-level Claude Code skills directory:

```sh
cp -r .claude/skills/* <your-project>/.claude/skills/
```

## Repository layout

```
.claude/
  settings.json            # permissions allowlist
  skills/
    smart-status/SKILL.md
    smart-branch/SKILL.md
    smart-save/SKILL.md
    smart-commit/SKILL.md
    smart-pull-request/SKILL.md
    smart-merge/SKILL.md
    smart-cleanup/SKILL.md
```

## Adding a skill

Create `.claude/skills/<name>/SKILL.md` starting with YAML frontmatter, then follow the
structure used by the existing skills. See [CLAUDE.md](CLAUDE.md) for the authoring
conventions (frontmatter, phase layout, conventional commits, explicit staging, protected
branches).

```markdown
---
name: <kebab-case-name>
description: <what it does + when to use it>
---

# <Name> Skill
...
```
