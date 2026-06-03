---
name: start-project
description: The single entry point for a new project. Interviews the human (via the Product Manager) to produce a PRD, creates a dedicated GitHub repo for the project, registers it in the control-plane registry, assigns the team, and seeds the first issues. Use when starting a brand-new project from a plain-English intent.
---

# Start Project Skill

The kickoff workflow for the agent team. Owned by the **Orchestrator**
(`.claude/agents/orchestrator/persona.md`): it runs discovery, stands up the project's **own**
GitHub repo (the data plane), records it in this repo's registry (the control plane), assigns the
initial team, and seeds the first issues — then hands off to the lifecycle.

**Read first:** `docs/DESIGN.md` (§2 control/data plane, §6 state, §7 lifecycle) and `docs/gates.md`.

**Dependencies:** `git`; `gh` (GitHub CLI) authenticated.

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

### Phase 2: Discovery interview (delegate to Product Manager)

The Orchestrator does **not** interview the human itself — it delegates to the **Product Manager**
agent, whose job is requirements discovery.

- Invoke the `product-manager` agent to run the kickoff interview.
- It elicits: the problem, target users, goals & **non-goals**, key scenarios, constraints,
  success metrics, and the v1 scope.
- **Output:** a draft PRD following `docs/templates/prd-template.md` with **testable** acceptance
  criteria, a one-line summary, and a proposed project name.

### Phase 3: Confirm name & create the registry entry

- Derive a **kebab-case slug** from the project name. Validate: lowercase, hyphens only, starts
  with a letter, ≤ 40 chars.
- Ensure it's free: no existing `docs/projects/<slug>.md`, and `gh repo view <org>/<slug>` returns
  not-found. On collision, propose an alternate or offer to resume the existing project.
- Create `docs/projects/<slug>.md` from `docs/projects/_template.md`, filling the frontmatter
  (`slug`, `repo` placeholder, `status: proposed`, `phase: discovery`, `created`, `team:
  [orchestrator, product-manager]`) and the **Summary**.

### Phase 4: Create the project repo (data plane)

```bash
# Private by default — confirm before creating a public repo
gh repo create <org>/<slug> --private --description "<summary>"
# Clone OUTSIDE this repo's tree
git clone https://github.com/<org>/<slug> <workspaceRoot>/<slug>
```

Seed the new repo (in `<workspaceRoot>/<slug>`, staging files **by name**):
- `docs/PRD.md` — the PRD from Phase 2.
- `CLAUDE.md` — a short note that this is a **data-plane project** run by the team in the PM
  control-plane repo, linking back to it.
- Initial commit on `main` (conventional): `docs: add PRD and project scaffold`.

Then update the registry entry's `repo` (and later `board`) fields.

### Phase 5: Set up GitHub-native state & seed issues

State lives in the **project** repo (see DESIGN §6).

- **Labels:** `role:*` (one per assigned role), `status:*` (`blocked`, `in-progress`, `review`),
  `phase:*` (`discovery`…`release`).
- **Milestones:** one per phase — Discovery, Design, Build, Test, Review, Release.
- **Board (optional):** a GitHub Project with columns = phases.
- **Seed issues** (minimal and dependency-ordered, from `docs/templates/ticket-template.md`):
  1. `Review & approve PRD` — the gate-1 human checkpoint.
  2. `Architecture & ADRs` — `role:software-architect`, depends on #1.
  3. `Plan & break down into tickets` — `role:project-manager`, depends on #2.

  Do **not** pre-seed the whole backlog — the Project Manager grows it after architecture.

### Phase 6: Human gate — PRD approval (gate 1)

- Present the PRD summary + acceptance criteria for **human sign-off** (gate 1, `docs/gates.md`).
- **Approved:** set registry `status: active`, `phase: design`; hand off to the Architect.
- **Changes requested:** keep `status: proposed`; the Product Manager revises and re-presents.

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
  Seeded:   3 issues (PRD review, architecture, planning)
  Gate 1:   awaiting PRD approval

Next: on PRD approval → Software Architect (design phase).
```

## Rules

### Never:
- Create the project **inside** this repo's git tree (no nested repos).
- Use `git add .` / `git add -A` — stage by name in both repos.
- Skip the PRD or its human sign-off (gate 1).
- Push project code into the control-plane repo — it holds only the registry entry.
- Create a public repo without explicit confirmation.

### Always:
- Delegate the discovery interview to the **Product Manager**.
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
- **Human declines the PRD:** keep `status: proposed`; the Product Manager iterates.
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
