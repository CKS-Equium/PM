# Task decomposition: <ticket #N — title>

> Written by the **Senior Software Engineer** on the ticket branch as `docs/tasks/<ticket>.md` when a
> ticket splits into **three or more independent atomic tasks** (DESIGN §4 "Delegation depth").
> The **Orchestrator** spawns one Junior per task from this file; the Senior (or, if it is gone, a
> fresh Senior) integrates from this file plus the task branches. Every task must be buildable by a
> fresh agent with **no other context** — that is exactly who reads it.

- **Ticket:** #N · **Ticket branch:** `<type>/<kebab>` · **Contracts:** <ADR / interface refs the tasks implement>
- **Integration plan:** <how the pieces come together; what the Senior checks after merging the task branches>
- **Shared files nobody may touch:** <build config, module index, lockfiles … — the Senior edits these at integration>

## Task 1 — <imperative title>

- **Branch:** `task/<ticket>-1` (off the ticket branch; PR back into the ticket branch)
- **Scope (in):** <exactly what to build>
- **Scope (out):** <what looks related but is not this task>
- **Files it may touch:** <explicit list; anything else is out of scope>
- **Interface it implements:** <signature / contract ref>
- **Acceptance criteria:** <testable, one per line>
- **Tests it must add:** <named cases, incl. the failure-mode cases that apply>
- **Depends on:** none *(tasks in a fan-out must be independent — if this is not "none", it is not a fan-out)*

## Task 2 — …

<!-- Status is tracked on the ticket issue, not here. If a Junior returns a gap, the Senior revises
     the task entry and the Orchestrator re-spawns; the file stays the single source of truth. -->
