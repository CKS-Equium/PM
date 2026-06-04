---
name: run-project
description: Drives an active project through the lifecycle — Design → Build → Test → Review → Release → Post-mortem — by delegating each phase to the right agents and enforcing the gates in docs/gates.md. Use to advance or continue an in-flight project after its PRD is approved.
---

# Run Project Skill

The Orchestrator's execution loop. Where `start-project` stands a project up (through PRD
approval, gate 1), **run-project** carries it the rest of the way: it reads the project's state,
delegates the current phase to the owning agent(s), checks the gate, pauses for human sign-off
where required, advances, and repeats — finishing with the post-mortem.

Owned by the **Orchestrator** (`.claude/agents/orchestrator/persona.md`). The Orchestrator never
does phase work itself; it delegates and enforces gates.

**Read first:** `docs/DESIGN.md` (§3 roles, §7 lifecycle), `docs/gates.md` (every gate's DoD),
and the project's registry entry `docs/projects/<slug>.md`.

**Dependencies:** `git`; `gh` (GitHub CLI) authenticated.

## Phase → owner → gate map

| Phase | Delegate to | Produces | Exit gate (DoD in gates.md) |
|-------|-------------|----------|------------------------------|
| Design | `software-architect` (ADRs, contracts, NFRs); `ux-designer` + `ui-designer` if there's a UI | ADRs, interface contracts, UX/UI specs | **Gate 2** 👤 architecture + design approved |
| Plan | `project-manager` | Milestones + dependency-ordered tickets (GitHub Issues) | (part of gate 2) |
| Build | `senior-software-engineer` (decomposes → `junior-software-engineer`) | Implemented features (PRs) | **Gate 3** all tickets built, self-reviewed |
| Test | `quality-engineer` | Test plan + automated tests + results | **Gate 4** criteria verified, coverage met |
| Review | `reviewer-critic` + `security-engineer` | Review verdicts, security findings | **Gate 5** no unresolved high-severity findings |
| Release | `devops-release-engineer` (+ `technical-writer` for docs) | Release, changelog, docs | **Gate 6** 👤 CI green, deploy + rollback known |
| Post-mortem | `process-engineer` (via `run-postmortem`) | `docs/postmortems/<slug>.md` | **Gate 7** recommendations recorded & routed |

👤 = human sign-off required (DESIGN §7).

## Workflow

### Phase 0: Load state

```bash
# Registry entry (control plane)
cat docs/projects/<slug>.md
# Project board / open work (data plane)
gh issue list --repo <org>/<slug> --state open
gh pr list --repo <org>/<slug>
```

Determine the **current phase** (registry `phase:` + open issues/milestones) and the **next gate**
to clear. Confirm gate 1 (PRD) is already approved; if not, this is a job for `start-project`.

### Phase 1: Delegate the current phase

Invoke the owning agent(s) from the map above (via the Task tool), passing the named inputs they
expect (PRD, ADRs, specs, tickets — whatever their persona's *Inputs* lists). Run independent
work concurrently (e.g. UX and UI specs), serialize dependencies (Plan needs the architecture).

**Require progress narration** (DESIGN §6 "progress narration"): instruct each delegated agent to
post a *start* comment on its issue **and set `status:in-progress`**, short *milestone/decision/
blocker* comments as it works, and a **substantive done** comment (what changed + AC/PR refs) —
not "done per PR". **Agents whose tools lack `gh` (designers, Researcher, Product/Technical Writer)
return their narration text and you post it for them.** You (the Orchestrator) post
**phase/gate-transition** comments on the relevant issues. Signal, not noise. This is what gives the
dashboard a live story.

### Phase 2: Check the gate

Verify the phase's **Definition of Done** in `docs/gates.md`.
- **Met:** proceed to Phase 3.
- **Not met:** return the work to the **owning** phase with the specific gap. If the gap traces
  upstream (e.g. a Review finding rooted in a bad requirement), escalate to that earlier phase.

### Phase 3: Human gate (if this is gate 2 or 6)

**Raise it on the board first** so it surfaces in the dashboard's "Needs you" panel (no dashboard
change needed — the panel already shows open `needs-human` issues): open a **`needs-human` issue in
the project repo**, assigned to the operator, titled e.g. `🚦 Gate 2: approve architecture & design`,
body = the summary + links to the artifacts being approved.

Then present the artifacts for **human sign-off** (architecture at gate 2; release at gate 6).
- **Approved:** **close the gate issue** (clears it from "Needs you"), continue, and log the approval
  in the registry's Decision log.
- **Changes:** keep the gate issue open; return to the owning agent; re-present.

### Phase 4: Advance

- Update the registry entry: bump `phase:`, update **Current state**, append any decisions.
- Update GitHub state: advance the board, set the next milestone, and **close completed issues
  programmatically** (a `gh issue close` loop) — do **not** rely on PR-body `Closes #N` keywords;
  they're unreliable for multiple issues.

### Phase 5: Loop / finish

Repeat Phases 1–4 for the next phase. After **gate 6 (Release)** passes, invoke the
**`run-postmortem`** skill to close gate 7 — the project is not "done" until the post-mortem
is recorded.

## Rules

### Never:
- Do phase work in this skill — always delegate to the owning agent.
- Let an agent edit an artifact it doesn't own (enforce persona scopes).
- Pass a gate with its DoD unmet, unless **consciously waived** with a reason logged in the registry.
- Skip the human gates (architecture, release) or the post-mortem.
- Push project code into the control-plane repo.

### Always:
- Drive from `docs/gates.md` — the gate DoD is the source of truth for "done".
- Keep the registry entry and the GitHub board in sync with reality after every advance.
- Run independent phase work concurrently; serialize only true dependencies.
- Return failed gates to the owning phase (or upstream if rooted there).
- Require delegated agents to **narrate progress** on their issues (start + `status:in-progress`,
  milestones, substantive done); post phase/gate transitions yourself (DESIGN §6).

### Prefer:
- Small, verifiable increments over big-bang phases.
- Escalating ambiguity to the human rather than guessing across a gate.

## Edge Cases

- **PRD not yet approved:** stop — use `start-project` (gate 1) first.
- **Gate fails repeatedly:** escalate to the human with the blocking finding; consider a waiver.
- **Blocked on a dependency:** mark the issue `status:blocked`, surface it, work other ready phases.
- **Scope change mid-flight:** route back to the Product Manager (PRD) and re-baseline the plan.
- **No UI in scope:** skip UX/UI delegation in the Design phase.
- **Project paused:** set registry `status: paused`; resume by re-running this skill.

## Invocation

User can trigger with:
- "run project <slug>"
- "advance <slug>"
- "continue the project"
- "move <slug> to the next phase"
- "/run-project <slug>"
