---
name: project-status
description: Portfolio overview for the control plane — lists every registered project with its status, phase, next gate, and blockers (from the registry and, for active projects, GitHub), and regenerates the docs/projects/README.md index. The control-plane analog of smart-status. Use to see where all projects stand or to refresh the registry index.
---

# Project Status Skill

The portfolio view across every project the team is running. Where `smart-status` reports on one
git repo, **project-status** reports on the whole control plane: it reads each registry entry,
optionally enriches active projects with live GitHub state, and keeps the registry index current.

Owned by the **Orchestrator** (`.claude/agents/orchestrator/persona.md`).

**Reads:** `docs/projects/*.md` (excluding `_template.md`); optionally `gh` for live board state.
**Writes:** `docs/projects/README.md` (the index table) only.

**Dependencies:** `git`; `gh` (optional — for live issue/PR/board status).

## Workflow

### Phase 1: Read the registry

Read the frontmatter of every `docs/projects/<slug>.md` (skip `_template.md` and `README.md`):
`slug`, `status`, `phase`, `created`, `repo`, `team`, `board`, plus the **Current state** section.

### Phase 2: Enrich active projects (optional, if `gh` available)

For each `status: active` project, pull live signals from its repo:

```bash
gh issue list --repo <org>/<slug> --state open --json number,title,labels
gh pr list   --repo <org>/<slug> --json number,title,isDraft
```

Derive: open-issue count, anything labelled `status:blocked`, open PRs, and the **next gate** from
the current phase (per `docs/gates.md`).

### Phase 3: Render the portfolio

```
📂 PROJECTS (3)

▶ customer-portal      active    phase: build     next: gate 3 (build complete)
    Repo: CKS-Equium/customer-portal · 4 open issues (1 blocked) · 2 PRs
    Blocked: #12 awaiting API contract

▷ marketing-site       proposed  phase: discovery next: gate 1 (PRD approval)
    Repo: — · awaiting PRD sign-off

✓ internal-dashboard   shipped   phase: release   post-mortem: recorded
    Repo: CKS-Equium/internal-dashboard
```

### Phase 4: Regenerate the index

Rewrite the **Projects** table in `docs/projects/README.md` from the frontmatter gathered in
Phase 1 — one row per project (Project · Status · Phase · Repo). Leave the rest of the file intact.

### Phase 5: Suggest the next action per project

Map state → action, e.g.:
- `proposed` → "awaiting PRD approval (gate 1)".
- `active` → "`run-project <slug>` to advance" (or "unblock #N" if blocked).
- `shipped` without a post-mortem → "`run-postmortem <slug>`".

## Rules

### Never:
- Write to anything except `docs/projects/README.md` — this is a read/report skill.
- Invent state — if `gh` is unavailable, report registry-only and say so.
- Modify a project's own repo.

### Always:
- Treat each registry entry's frontmatter as the source of truth for status/phase.
- Skip `_template.md` and `README.md` when scanning projects.
- Note when live (GitHub) enrichment was skipped (offline / no `gh`).

### Prefer:
- Most-recently-active projects first.
- A compact one-screen portfolio over exhaustive per-issue detail.

## Edge Cases

- **No projects yet:** report an empty portfolio and point to `start-project`.
- **`gh` not authenticated / offline:** registry-only view; mark enrichment as skipped.
- **Registry/GitHub drift** (e.g. registry says build, board says review): flag the mismatch for
  the Orchestrator to reconcile — don't silently pick one.
- **Malformed frontmatter:** report the file as needing repair rather than skipping it silently.

## Invocation

User can trigger with:
- "project status"
- "portfolio"
- "list projects"
- "where do the projects stand"
- "refresh the project index"
- "/project-status"
