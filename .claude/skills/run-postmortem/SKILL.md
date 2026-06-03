---
name: run-postmortem
description: Facilitates the closing post-mortem after a project ships — each agent self-reviews, then 360-reviews the chain neighbors it worked with; recommendations route to notes.md (free) or persona.md (review-gated). Produces docs/postmortems/<slug>.md. Use after release to close the project and feed the self-improvement loop.
---

# Run Post-mortem Skill

The closing gate (gate 7) and the engine of the team's recursive self-improvement. A project is
not "done" until its post-mortem is recorded. Facilitated by the **Process Engineer**
(`.claude/agents/process-engineer/persona.md`); triggered by `run-project` after release.

**Read first:** `docs/DESIGN.md` (§5 self-improvement, §7 post-mortem), `docs/gates.md` (gate 7),
the project registry entry, and `docs/postmortems/_template.md`.

## Preconditions

- The project has shipped — **gate 6 (Release)** passed.
- The registry entry lists the `team` that participated (the agents to review).

## Workflow

### Phase 1: Stage 1 — self-review (each agent)

Each participating agent assesses its own performance on this project: what it **kept** (worked
well) and what it would **drop / change**. Each lesson is appended to that agent's **own**
`notes.md` (the unrestricted self-edit path — no gate), in the entry format
*situation → what happened → lesson → confidence*.

### Phase 2: Stage 2 — 360 review (chain neighbors only)

Each agent reviews the collaborators **up and down its org chart that it actually worked with** —
**not** all-to-all (that's O(n²) noise; see DESIGN §7). For example: the Senior Engineer reviews
the Architect (up) and the Juniors (down); it does not review the UI Designer it never touched.

Each review captures: what worked, and one concrete friction/suggestion.

### Phase 3: Aggregate recommendations (Process Engineer)

The Process Engineer compiles all 360 feedback into a deduplicated recommendation list, tagging
each with its **level**:
- **notes-level** — a heuristic/gotcha for one role's playbook.
- **contract-level** — a change to a role's scope (its `persona.md`).

### Phase 4: Route the recommendations

- **notes-level** → append to the **target** agent's `notes.md` (free).
- **contract-level** → open a **review-gated PR** against the target's `persona.md`; the
  **Reviewer/Critic** (or human) approves before merge (DESIGN §5, the contract gate).

### Phase 5: Write the report

Produce `docs/postmortems/<slug>.md` from `docs/postmortems/_template.md`: self-assessments, 360
reviews, the recommendation table (recommendation / target role / level / severity / owner), and
**Actions taken** linking each recommendation to its `notes.md` commit or contract PR — so it can
later be audited whether the team acted on what it learned.

### Phase 6: Close gate 7

- Update the registry: `status: shipped` (or `archived`), final **Current state**.
- Regenerate the post-mortems index (`docs/postmortems/README.md`).
- Commit in the control-plane repo (stage explicitly).

## Rules

### Never:
- Skip the post-mortem — it is the project's closing gate and the team's main learning signal.
- Edit a `persona.md` contract directly — contract changes go through the reviewed-PR gate.
- Run an all-to-all 360 — scope each agent's review to chain neighbors it worked with.
- Lose the link between a recommendation and the action taken (or explicitly not taken).

### Always:
- Run self-review (Stage 1) before the 360 (Stage 2).
- Record self-lessons in each agent's own `notes.md`.
- Tag every recommendation as notes-level or contract-level and route accordingly.
- Capture the metrics signal (gates passed first try, coverage, review findings, rework loops).

### Prefer:
- Concrete, assigned recommendations over vague praise/blame.
- Promoting only repeatedly-proven heuristics from notes into contracts.

## Edge Cases

- **Project cancelled (not shipped):** still run a lightweight post-mortem — why it stopped is a
  high-value lesson. Mark `status: archived`.
- **Solo-ish project (few roles):** Stage 2 may be small or skipped; Stage 1 still runs.
- **Contract PR rejected by the Reviewer:** keep the recommendation as a notes-level entry instead;
  record the rejection reason in the report.
- **No clear signal/metrics:** note the gap — improving measurement is itself a recommendation.

## Invocation

User can trigger with:
- "run postmortem <slug>"
- "post-mortem for <slug>"
- "close out <slug>"
- "retro <slug>"
- "/run-postmortem <slug>"
