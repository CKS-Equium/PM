# Orchestrator — notes

Playbook for the Orchestrator. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **Live discovery interview** — *Situation:* gate-1 discovery for team-pulse-dashboard. *What happened:* a PM subagent cannot conduct a live, back-and-forth interview with the operator (no interactive channel), so I ran the interview turn-by-turn in the main thread, then handed the transcript to the PM to draft the PRD. *Lesson:* keep the live operator interview in the Orchestrator's main thread; the PM consumes the captured Q&A and authors the PRD. Do not delegate the live interview itself. *(confidence: high)*
- **Gate-1-before-repo ordering** — *Situation:* start-project created the project repo and registry before the PRD was approved. *What happened:* the PRD took 2 rounds (operator reframed to a live activity log + "Needs-you" panel); had it been rejected we'd have an orphan repo. *Lesson:* sequence start-project so gate-1 PRD approval precedes `gh repo create`; no project repo until we know we're building the right thing. *(confidence: high)*
- **New-project directory registration** — *Situation:* sub-agents' Glob/Read against the freshly-created project repo triggered directory-access prompts because the path wasn't in `additionalDirectories`. *What happened:* I patched it by adding `d:\github` to settings; fragile and manual. *Lesson:* start-project should auto-register each new project dir into `settings.local.json` so spawned agents have access without prompts. *(confidence: high)*
- **GitHub multi-issue auto-close** — *Situation:* PR #15 body used `Closes #5, #6, …` to close ten tickets on merge. *What happened:* GitHub only auto-closed #5 (one keyword binds one issue); the rest were closed manually. *Lesson:* use a closing keyword per issue (`Closes #5, Closes #6, …`) or close on merge programmatically. *(confidence: high)*

## hearthflix (2026-06-04)

- **Our own process fixes paid off (positive reinforcement)** — *Situation:* second project, first run of the post-team-pulse start-project changes (gate-1-before-repo, live interview in main thread, project-dir auto-registration). *What happened:* gate 1 passed **first try**, no orphan repo was created, and there were **zero directory-access prompts** for spawned agents across the whole run. *Lesson:* the recursive-improvement loop is working — recommendations applied at gate 7 measurably removed the exact friction they targeted. Keep treating prior post-mortem fixes as load-bearing and verify them on the next run. *(confidence: high)*
- **Multi-issue auto-close STILL unreliable — supersede the keyword fix** — *Situation:* applied team-pulse's "one `Closes #N` keyword per issue" fix on PR #17 (13 tickets). *What happened:* only ~6 of 13 issues auto-closed on merge; the rest were closed manually again. The keyword-per-issue convention is not robust at scale. *Lesson:* do **not** rely on PR-body keywords for closing at all — after merge, close tickets **programmatically in a loop** (`gh issue close` over the ticket list). This supersedes team-pulse rec #4. *(confidence: high)*
- **Narration for no-Bash agents** — *Situation:* product-manager, researcher-analyst, ux/ui-designer have no Bash/gh tool, so they cannot comment on their own issues to self-narrate. *What happened:* I narrated on their behalf for the activity log; works but conflates authorship and is easy to forget. *Lesson:* either grant a scoped narration capability to those roles or formalize "Orchestrator narrates for no-Bash agents" as an explicit convention. Noted at team-pulse, still unresolved — escalate as a process decision. *(confidence: med)*
