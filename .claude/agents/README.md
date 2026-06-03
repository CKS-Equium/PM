# Agents — the team

Each role in the team (see [docs/ORG.md](../../docs/ORG.md)) is a directory here:

```
.claude/agents/<role>/
  persona.md   # the CONTRACT — mission, scope, does-NOT. Stable, REVIEW-GATED.
  notes.md     # the PLAYBOOK — accumulating heuristics. Agent self-appends FREELY.
```

## The two files

- **`persona.md`** is the agent's system prompt and scope contract. It carries Claude Code agent
  frontmatter (`name`, `description`, `tools`, `model`) plus the contract body. It changes
  **only via a reviewed PR** (Reviewer/Critic or human approves) — never edited silently. This
  is what keeps each agent in its lane.
- **`notes.md`** is the agent's growing playbook. The agent appends to its **own** notes freely
  (no gate). Entry shape: *situation → what happened → lesson → confidence.* The Process Engineer
  periodically distills proven notes into the contract (through the gate) and prunes stale ones.

See [DESIGN.md](../../docs/DESIGN.md) §5 for the full self-improvement model and §8 for model
tiering.

## Contract template

New personas start from [`docs/templates/persona-template.md`](../../docs/templates/persona-template.md).
(The template lives under `docs/templates/`, not here, so it isn't loaded as a real agent.)

## The 15 roles

`orchestrator` · `process-engineer` · `product-manager` · `project-manager` ·
`researcher-analyst` · `software-architect` · `senior-software-engineer` ·
`junior-software-engineer` · `quality-engineer` · `reviewer-critic` · `security-engineer` ·
`ux-designer` · `ui-designer` · `devops-release-engineer` · `technical-writer`

> All 15 personas exist (round 1), authored against [DESIGN.md](../../docs/DESIGN.md) §3, which
> remains the summary view of every role's scope.
