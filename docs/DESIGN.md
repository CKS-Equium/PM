# DESIGN — Autonomous Multi-Agent Project Management Team

> **Status:** Rev 1 blueprint. This is the single source of truth for the architecture.
> Personas and skills are built against this document; if they diverge, this wins until
> consciously revised.

## 1. Purpose

Build an **autonomous, multi-agent software-project-management team** — a set of role-based
agents with strict, non-overlapping scopes that can take a software project from a plain-English
intent through to a released, documented product, and get measurably better with every project
it runs.

This repository is the **team**, not any one project (see §2).

## 2. Control plane vs data plane

The single most important architectural decision: separate the reusable *team* from the
*projects* it executes.

| | **Control plane** (this repo) | **Data plane** (per project) |
|---|---|---|
| What | The team: personas, skills, templates, accumulated learning | The actual product being built |
| Storage | This GitHub repo | A **separate GitHub repo per project**, created on demand |
| Lifetime | Permanent, reusable across all projects | One per project |
| Tracked here | Only a **registry** entry linking to the project repo (`docs/projects/<slug>.md`) | All code, artifacts, project-specific context |

**Why:** the team — with everything it has learned — gets pointed at project after project.
Durable lessons flow back into the control plane; project-specific context stays in the project
repo. The team is portable and could even be templated/published.

**Mechanics:** project repos live *outside* this repo's git tree (nesting git repos is messy).
The Orchestrator runs `gh repo create` and works in a clone elsewhere; this repo only stores the
registry entry. Requires `gh` (GitHub CLI), authenticated.

## 3. The org (15 roles)

Hierarchical delegation: **Human → Orchestrator → leads → workers.** Full chart in
[ORG.md](ORG.md).

| Layer | Role | Owns / writes | Must NOT |
|---|---|---|---|
| Governance | **Orchestrator** (Delivery Lead) | Routing, project state, gate enforcement, escalation | Make product/architecture/code decisions |
| Governance | **Process Engineer** | The SDLC itself: gates, DoD, metrics, retros; improves personas | Do the project's actual work |
| Product | **Product Manager** | Vision, PRD, user stories, acceptance criteria, priority; kickoff interview | Decide tech or schedule |
| Product | **Project Manager** | Plan, milestones, tickets, dependencies, risk, status | Define requirements or implementation |
| Product | **Researcher / Analyst** | Spikes, option/library evaluations, findings | Make the final call (informs, doesn't decide) |
| Engineering | **Software Architect** (Tech Lead) | System design, ADRs, interface contracts, NFRs, stack | Write feature code or own schedule |
| Engineering | **Senior Software Engineer** | Complex, multi-step features; decomposes into atomic task specs (the Orchestrator spawns the Juniors) | Change architecture without Architect sign-off; spawn agents |
| Engineering | **Junior Software Engineer** | One atomic, fully-specified task | Make design decisions / touch unrelated code |
| Quality | **Quality Engineer** (QA) | Test strategy, test cases, automated tests, verification | Write feature code |
| Quality | **Reviewer / Critic** | Adversarial review of diffs & artifacts | Author code (reviews only) |
| Quality | **Security Engineer** | Threat modeling, security review, dependency/supply-chain | Own functional correctness |
| Design | **UX Designer** | Flows, IA, interaction, accessibility, wireframes | Visual styling or implementation |
| Design | **UI Designer** | Visual design, components, design system, tokens | Define flows or write app logic |
| Delivery | **DevOps / Release Engineer** | CI/CD, environments, build/deploy, observability, releases | Write feature code |
| Delivery | **Technical Writer** | User docs, READMEs, changelogs, API docs | Decide product or code |

**Independence:** Quality / Reviewer / Security report to the Orchestrator, **not** to the
engineers they review. The Process Engineer is a staff function (advises the Orchestrator,
improves every persona) — it is not in the delivery chain.

**Minimum team.** The 15 roles are the org, not the roster for every project. The Orchestrator
assigns the **core six** by default — Orchestrator, Product Manager, Software Architect, Senior
Engineer, Quality Engineer, Reviewer/Critic — and adds roles only when the project has the surface
they own: Security Engineer (network / untrusted input / dependencies), UX + UI Designer (a user
interface), Project Manager (more than ~10 tickets or multiple milestones), Researcher (an open
technical question that gates a decision), DevOps (a deploy target beyond "run it locally"),
Technical Writer (an external audience for docs), Junior Engineers (a fan-out, §4). Record the
assigned team in the registry entry. **An unassigned role's DoD items do not vanish:** tickets →
the Senior writes them from the Architect's plan; the gate-5 security review → the Reviewer/Critic
runs the Security Engineer's standard checks; the gate-6 release → the Senior authors the release artifacts and the Orchestrator runs the
git/release skills (it authors nothing); docs →
the Senior. Anything else is a logged waiver.

## 4. Substrate & file layout

Each role is a **Claude Code agent** — a directory under `.claude/agents/<role>/`:

```
.claude/
  agents/<role>/
    persona.md     # the CONTRACT — mission, scope, does-NOT. Stable, review-gated.
    notes.md       # the PLAYBOOK — accumulating heuristics. Agent self-appends freely.
  skills/          # workflow skills (incl. the existing git lifecycle skills)
  settings.json
docs/
  DESIGN.md        # this file
  ORG.md           # org chart
  gates.md         # phase gates, DoD, human approval points
  templates/       # artifact templates (PRD, ADR, ticket, task-decomposition, test-plan, UX/UI spec)
  projects/        # registry — one <slug>.md per project + auto-generated index
  postmortems/     # one <slug>.md 360-review report per completed project
```

### Delegation depth: one level

Claude Code subagents **cannot spawn subagents**, so the only agent that delegates at runtime is the
Orchestrator (the main session). The Senior → Junior chain in §3 is therefore a **fan-out through the
Orchestrator**, not a nested call:

1. The Orchestrator invokes the Senior with a ticket. If the ticket splits into **three or more
   independent atomic tasks**, the Senior writes the decomposition to the branch
   (`docs/tasks/<ticket>.md`, from `docs/templates/task-decomposition-template.md`) and returns
   without building. The file names the shared files no task may touch.
2. The Orchestrator spawns **one Junior per task, in parallel**, each with only its task spec and the
   contracts it needs, each on its own branch `task/<ticket>-<n>` off the ticket branch.
3. The Orchestrator sends the Juniors' results back to the **same Senior agent** (continued, so it
   keeps its plan) for integration, self-review, and the gate-3 checklist. If that agent is gone, a
   fresh Senior integrates from the decomposition file + the task branches — the file must make that
   possible (the interruption-tolerance rule in gates.md). A Junior that returns a gap goes back to
   the Senior for a revised entry; only that task is re-spawned.

Below the fan-out threshold the Senior simply builds the ticket itself; the Junior tier exists for
parallelism and cost, not capability. The decomposition file is the handoff artifact — it must be
readable by a fresh agent, because that is exactly who reads it.

### Persona contract template

`persona.md` carries Claude Code frontmatter (`name`, `description`, `tools`, `model`) plus:
**Mission · Perspective · Owns · Does NOT do · Inputs · Outputs · Handoffs · Definition of
Done · Escalation · Tools/permissions.** Template: [docs/templates/persona-template.md](templates/persona-template.md).

## 5. Self-improvement (recursive)

The team improves itself, deliberately and auditably:

- **`notes.md` — unrestricted.** Any agent appends learnings to its own playbook at any time.
  Entry shape: *situation → what happened → lesson → confidence.*
- **`persona.md` — gated.** Changing a scope contract requires a **reviewed PR** (Reviewer/Critic
  or human approves). No silent self-editing of contracts; enforced by convention + review (no hook).
- **Process Engineer = distiller.** Periodically promotes proven `notes.md` heuristics into the
  `persona.md` contract (via the gate) and prunes contradictory/stale notes.
- **Git is the ledger.** Every improvement is a commit — diffable, attributable, revertible.
- **Signal.** Improvement needs evidence of "what worked." The primary signal is the **post-mortem**
  (§7), supplemented by QA pass/fail, Reviewer verdicts, and gate-pass-first-try rates.

## 6. State tracking — GitHub-native

State lives in each project's GitHub repo, no custom store required to start:

| State concept | GitHub primitive |
|---|---|
| Ticket / task | **Issue** (body = spec) |
| Owner / status / gate | **Labels** (`role:senior-dev`, `status:blocked`, `phase:design`) |
| Live board (Orchestrator's view) | **GitHub Project** (columns = phases) |
| Phase / release | **Milestone** |
| Handoff log + agent activity | **Issue comments** |
| Artifact delivery | **PR**, linked to its issue |

### Progress narration (the live story)

Agents **narrate their work on the issue** so the activity log reads as a live story, not just
open→close. Per work item:
- **On pickup:** comment the approach and apply `status:in-progress` (this also populates the
  in-progress event class the dashboard tracks).
- **At milestones / decisions / blockers:** short comments (a blocker becomes / links a
  `needs-human` item).
- **On completion:** a *substantive* closing comment — what changed and why, with AC/PR refs —
  never a bare "done per PR#15".

The Orchestrator additionally comments **phase/gate transitions** on the relevant issues, and
**posts narration on behalf of agents whose tools lack `gh`** (designers, Researcher,
Product/Technical Writer) — those agents return their narration text and the Orchestrator posts it.
Keep it **signal, not noise** (≈ start + 0–3 substance comments + a real close per issue); the bar is
"would someone watching the dashboard care?" The dashboard renders these `comment` events with no
app change.

A **custom dashboard is the team's first dogfood project** — it later adds *agent-ops*
observability (cost, model, logs, live activity) that GitHub can't show.

## 7. Lifecycle, gates & post-mortem

```
human intent
  └─ start-project (Orchestrator) ─ delegates interview to Product Manager
       │  creates project repo · registry entry · team · first issues
       ▼
   Discovery → Design → Build → Test → Review → Release → Post-mortem
   (Product)   (Arch/   (Senior  (QA)   (Reviewer  (DevOps)  (Process Eng)
               UX/UI)   →Junior)        +Security)
```

- **Gates & DoD** between every phase, owned by the Process Engineer — see [gates.md](gates.md).
- **Human-in-the-loop** at exactly two points: **brief approval** (gate 1) and **final acceptance**
  (gate 6 — release, or the playtest / operator-path run). Architecture is surfaced as a
  non-blocking `needs-human` FYI. Everything between runs autonomously with the Reviewer/Critic as
  the gate (BLOCK = fix-before-merge). See [gates.md](gates.md).
- **Post-mortem = the closing gate.** A project isn't "done" until it's recorded. Two stages:
  1. **Self-review** — each agent assesses itself (positive + negative) → its own `notes.md`.
  2. **360 review** — each agent reviews collaborators **up and down its chain** (scoped to
     neighbors it actually worked with) → improvement recommendations. Notes-level land in the
     target's `notes.md`; contract-level become review-gated `persona.md` PRs.
  Report: `docs/postmortems/<slug>.md`. This is the primary feedback signal for §5.

### Execution mode: running an operator-authored dev plan

The same lifecycle with Discovery and Design already done. When the operator supplies an existing
design **and** a gated dev plan, gate 1 confirms them as the brief and the team **executes** the
plan: the **project's own per-gate Definition of Done** stands in for gates 2–5, the Reviewer/Critic
is the gate on every PR (**BLOCK = fix-before-merge**), and the run proceeds autonomously until
**plan-complete-needs-playtest** or a hard blocker — **branch + PR per gate, self-merged on a green
DoD, gate branches kept as checkpoints**. Gate 6 is the **final playtest / acceptance**. Entry point:
`start-project` with a design + plan instead of an intent (it skips the interview); `run-project`
detects the plan and drives from it. First instance: **colonygame**.

## 8. Model tiering

Set per agent via the `model:` frontmatter; tunable per project. *(Revised 2026-09-09.)*

- **Fable** (highest judgment, reads everything): **Orchestrator** — independent gate verification,
  recovery when a delegate dies, the only agent that sees the whole board; **Reviewer/Critic** —
  adversarial review caught five shippable defects on new-cadair that green suites missed, and it
  reads a diff, not a
  project, so it is the cheapest place to buy correctness.
- **Opus** (senior judgment): Process Engineer, Product Manager, Software Architect, Senior Software
  Engineer.
- **Sonnet:** Project Manager, Researcher/Analyst, Quality Engineer, Security Engineer, UX Designer,
  UI Designer, DevOps/Release Engineer, Technical Writer, **Junior Software Engineer** (atomic tasks
  are rarely as atomic as a Haiku tier assumed; Sonnet asks rather than guesses).

The Orchestrator is normally the **main session**, so its frontmatter only applies when it is spawned
as a subagent — in practice "Orchestrator = Fable" means running the session on Fable.

## 9. Build order

1. **Scaffold** — this design doc + ORG, gates, templates, registry, post-mortem skeleton. *(done)*
2. **Personas** — the 15 `persona.md` contracts + empty `notes.md`. *(done)*
3. **Orchestration** — `start-project` (kickoff) + `run-project` (lifecycle/gate) + `run-postmortem`
   (closing gate). *(done)*
4. **Project spin-up & visibility** — repo/registry/issue seeding (in `start-project`) +
   `project-status` (portfolio view + registry-index regeneration). *(done)*
5. **First project** — build the dashboard (dogfood), then run a real post-mortem. *(done —
   `team-pulse-dashboard`, 2026-06-03; six projects and seven post-mortems since.)*

## 10. Dependencies

`git`; `gh` (GitHub CLI, authenticated) for repo creation, issues, projects, and PRs;
Claude Code as the runtime. Per-project quality-gate tools (linters, test runners) are supplied
by each project, not this repo.
