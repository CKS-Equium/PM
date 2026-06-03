# Org Chart

Hierarchical delegation. The Human delegates to the Orchestrator; the Orchestrator delegates to
leads; leads delegate to workers. Results bubble back up through the same chain.

```
Human / Sponsor
└─ Orchestrator (Delivery Lead)        ┄┄┄ advised by → Process Engineer
   │                                          (retros, metrics, improves all personas)
   ├─ Product Manager — what & why
   │     └─ Researcher / Analyst — investigates unknowns, evaluates options
   │
   ├─ Software Architect — how (system design, ADRs, contracts)
   │     └─ Senior Software Engineer — complex, multi-step features
   │           └─ Junior Software Engineer ×N — atomic, specified tasks
   │
   ├─ Project Manager — plan, tickets, dependencies, risk, status
   │
   ├─ Design
   │     ├─ UX Designer — flows, IA, interaction, accessibility
   │     └─ UI Designer — visual design, components, design system
   │
   ├─ Quality & Safety  (independent review gate)
   │     ├─ Quality Engineer — test strategy & verification
   │     ├─ Reviewer / Critic — adversarial review
   │     └─ Security Engineer — threat modeling, security review
   │
   └─ Delivery & Knowledge
         ├─ DevOps / Release Engineer — CI/CD, releases (owns the git skills)
         └─ Technical Writer — docs, changelogs
```

## Reporting principles

- **Independence of review.** Quality Engineer, Reviewer/Critic, and Security Engineer report to
  the **Orchestrator**, never to the engineers whose work they assess. This keeps review honest.
- **Process Engineer is staff, not line.** It sits beside the Orchestrator (dotted line), advises
  on process, and owns the gates and the persona-improvement loop — it does no project work itself.
- **Senior vs Junior split = scope of decision.** The Senior Engineer decomposes a feature and
  spawns Juniors; a Junior executes exactly one atomic, fully-specified task and makes no design
  calls.
- **Delegation only flows down; escalation only flows up.** No agent reaches across the chart to
  another's lane — conflicts escalate to the nearest common parent (usually the Orchestrator).

See [DESIGN.md](DESIGN.md) §3 for each role's owns/must-not, and the per-role contracts under
[`.claude/agents/`](../.claude/agents/).
