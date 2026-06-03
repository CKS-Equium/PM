---
name: <role-kebab-case>
description: <one line — what this role does AND when it's invoked. Used for delegation routing.>
tools: <comma-separated allowlist, e.g. Read, Grep, Glob, Write, Edit, Bash>
model: <opus | sonnet | haiku>   # per DESIGN.md §8
---

# <Role Name>

**Mission:** <one sentence — the single thing this agent exists to do.>

**Perspective:** <the lens this agent reasons through — what it optimizes for, what it worries about.>

## Owns
- <Decisions this agent is authorized to make.>
- <Artifacts it may write (by path / type).>

## Does NOT do
- <Explicit out-of-scope — the most important section. What it must hand off rather than do.>

## Inputs
- <Artifacts / triggers it consumes, and from whom.>

## Outputs
- <Artifacts it produces, in what format, for whom.>

## Handoffs
- **Receives from:** <role(s) / artifact(s)>
- **Hands off to:** <role(s) / artifact(s)>

## Definition of Done
- <Its quality bar — what "finished" means for its outputs. Ties to the relevant gate in docs/gates.md.>

## Escalation
- <When to defer to the Orchestrator or a human rather than proceed.>

<!-- Learnings live in this role's notes.md, not here. Contract changes require a reviewed PR. -->
