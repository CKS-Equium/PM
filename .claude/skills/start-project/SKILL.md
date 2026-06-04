---
name: start-project
description: The single entry point for a new project. Interviews the human (via the Product Manager) to produce a PRD, gets it human-approved, then creates a dedicated GitHub repo for the project, registers it in the control-plane registry, assigns the team, and seeds the first issues. Use when starting a brand-new project from a plain-English intent.
---

# Start Project Skill

The kickoff workflow for the agent team. Owned by the **Orchestrator**
(`.claude/agents/orchestrator/persona.md`): it runs discovery, gets the PRD approved, stands up the
project's **own** GitHub repo (the data plane), records it in this repo's registry (the control
plane), assigns the initial team, and seeds the first issues — then hands off to the lifecycle.

**Read first:** `docs/DESIGN.md` (§2 control/data plane, §6 state, §7 lifecycle) and `docs/gates.md`.

**Dependencies:** `git`; `gh` (GitHub CLI) authenticated.

> **Ordering note (learned in the first dogfood):** the PRD is approved at **gate 1 _before_ the
> GitHub repo is created** — never stand up external resources for a PRD that may be rejected.

## Workflow

### Phase 1: Pre-flight

Confirm we're in the right place and have what we need.

```bash
# Must be the control-plane repo (the team), not a project
test -f docs/DESIGN.md && git remote -v
# gh must be authenticated
gh auth status
```

**Determine targets** (in priority order — config, then derive, then ask):
- **Org** for the new repo: `.startproject.json` `org` → else the owner of this repo's origin
  (`gh repo view --json owner -q .owner.login`) → else ask.
- **Workspace root** where project repos are cloned (must be **outside** this repo's tree):
  `.startproject.json` `workspaceRoot` → else the parent directory of this repo → else ask.

**Stop if:** `gh` isn't authenticated, or there's no org access. Never proceed without a place to
put the project repo that is outside this tree.

### Phase 2: Discovery interview → PRD

Requirements discovery is the Product Manager's job — but a **subagent cannot hold a live,
back-and-forth interview** with the human (subagents run autonomously and return a result). So the
Orchestrator runs the **interview turn-by-turn in its own main thread** (asking the PM's discovery
questions), then hands the captured answers to the `product-manager` agent to **draft the PRD from
that transcript**.

- Elicit: the problem, target users, goals & **non-goals**, key scenarios, constraints, success
  metrics, and the v1 scope. For an ops/tooling request, probe early: *what concrete action does
  this drive for the user?*
- The `product-manager` agent then produces a draft PRD following `docs/templates/prd-template.md`
  with **testable** acceptance criteria, a one-line summary, and a proposed project name.

### Phase 3: Confirm name & create the registry entry

- Derive a **kebab-case slug** from the project name. Validate: lowercase, hyphens only, starts
  with a letter, ≤ 40 chars.
- Ensure it's free: no existing `docs/projects/<slug>.md`, and `gh repo view <org>/<slug>` returns
  not-found. On collision, propose an alternate or offer to resume the existing project.
- Create `docs/projects/<slug>.md` from `docs/projects/_template.md`, filling the frontmatter
  (`slug`, `repo` placeholder, `status: proposed`, `phase: discovery`, `created`, `team:
  [orchestrator, product-manager]`) and the **Summary**.

### Phase 4: Human gate — PRD approval (gate 1)

Present the PRD summary + acceptance criteria for **human sign-off** (gate 1, `docs/gates.md`) —
**before any repo is created**.

- **Approved:** proceed to Phase 5. (The registry flips to `status: active` once the repo exists.)
- **Changes requested:** keep `status: proposed`; the Product Manager revises and re-presents. No
  external resources have been created yet, so iteration is cheap.

### Phase 5: Create the project repo (data plane)

```bash
# Private by default — confirm before creating a public repo
gh repo create <org>/<slug> --private --description "<summary>"
# Clone OUTSIDE this repo's tree
git clone https://github.com/<org>/<slug> <workspaceRoot>/<slug>
```

Seed the new repo (in `<workspaceRoot>/<slug>`, staging files **by name**):
- `docs/PRD.md` — the approved PRD from Phase 2.
- `CLAUDE.md` — a short note that this is a **data-plane project** run by the team in the PM
  control-plane repo, linking back to it.
- Initial commit on `main` (conventional): `docs: add PRD and project scaffold`.

Then update the registry entry's `repo` field and set `status: active`, `phase: design`.

**Register the new directory** so sub-agents can read/search the project repo without
per-directory access prompts: add `<workspaceRoot>/<slug>` to `permissions.additionalDirectories`
in `.claude/settings.local.json` (create the file if absent — it's git-ignored). Takes effect on
the next session.

### Phase 6: Set up GitHub-native state & seed issues

State lives in the **project** repo (see DESIGN §6).

- **Labels:** `role:*` (one per assigned role), `status:*` (`blocked`, `in-progress`, `review`),
  `phase:*` (`discovery`…`release`), and `needs-human` (agent → operator escalations).
- **Milestones:** one per phase — Discovery, Design, Build, Test, Review, Release.
- **Board (optional):** a GitHub Project with columns = phases (needs the `project` token scope).
- **Seed issues** (minimal and dependency-ordered, from `docs/templates/ticket-template.md`):
  1. `Architecture & ADRs` — `role:software-architect`.
  2. `Plan & break down into tickets` — `role:project-manager`, depends on #1.

  Do **not** pre-seed the whole backlog — the Project Manager grows it after architecture.
- **Issue-closing convention:** close delivered tickets **programmatically after merge** (a
  `gh issue close` loop), not via PR-body `Closes #N` keywords — those are unreliable for multiple
  issues (proven across two projects). Reference issues in the PR body for traceability only.

### Phase 7: Summary & handoff

Report what was created, then commit the control-plane changes here.

```bash
# In the control-plane repo, stage explicitly
git add docs/projects/<slug>.md docs/projects/README.md
git commit -m "chore: register project <slug>"
```

**Display:**
```
✅ Project started: <slug>
  Repo:     https://github.com/<org>/<slug>
  Registry: docs/projects/<slug>.md
  Board:    <url>
  Seeded:   issues (architecture, planning)
  Gate 1:   ✅ PRD approved

Next: Software Architect (design phase) → run-project <slug>.
```

## Rules

### Never:
- Create the project **inside** this repo's git tree (no nested repos).
- Create the GitHub repo **before** the PRD passes gate 1.
- Use `git add .` / `git add -A` — stage by name in both repos.
- Skip the PRD or its human sign-off (gate 1).
- Push project code into the control-plane repo — it holds only the registry entry.
- Create a public repo without explicit confirmation.

### Always:
- Run the discovery interview in the Orchestrator's main thread; the **Product Manager** drafts the
  PRD from the transcript.
- Approve the PRD (gate 1) before creating any external resources.
- Register the new project's directory in `.claude/settings.local.json` `additionalDirectories`.
- Keep each project in its **own** repo; this repo stores only metadata/links.
- Use conventional commits and respect protected branches (`main`/`master`/`develop`) in both repos.
- Record key decisions in the registry entry's Decision log.

### Prefer:
- Private repos by default.
- Deriving org/workspace from `.startproject.json` or the control-plane remote over asking.
- A minimal, dependency-ordered set of first issues over a full backlog.

## Edge Cases

- **`gh` not authenticated:** stop; instruct `gh auth login`.
- **Slug or repo already exists:** offer an alternate slug, or resume the existing project.
- **No org access:** ask which org or personal account to use.
- **Offline:** draft the PRD + registry entry locally; defer repo creation and note it in the entry.
- **Human declines the PRD:** keep `status: proposed`; the Product Manager iterates. No repo exists yet.
- **Workspace dir missing:** create it (confirming it's outside this repo's tree).

## Configuration

Optional `.startproject.json` in the control-plane repo:

```json
{
  "org": "CKS-Equium",
  "workspaceRoot": "..",
  "visibility": "private",
  "createBoard": true,
  "seedLabels": true
}
```

## Invocation

User can trigger with:
- "start project"
- "start a new project"
- "kick off <name>"
- "new project: <description>"
- "/start-project <description>"
