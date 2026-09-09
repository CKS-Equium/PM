# Project Registry

Each project the team executes lives in its **own GitHub repo** (the data plane). This directory
is the control-plane index of those projects — metadata and links only, never project code.

- One file per project: `docs/projects/<slug>.md` (markdown + YAML frontmatter).
- The frontmatter **is** the database — diffable in git, no external store.
- New entries are created by the `start-project` skill; the table below is regenerated from each
  file's frontmatter.
- Template: [`_template.md`](_template.md).

## Projects

| Project | Status | Phase | Repo |
|---------|--------|-------|------|
| [localcoder](localcoder.md) | active | build | [CKS-Equium/localcoder](https://github.com/CKS-Equium/localcoder) |
| [event-intel](event-intel.md) | active | release | [CKS-Equium/event-intel](https://github.com/CKS-Equium/event-intel) |
| [new-cadair](new-cadair.md) | shipped | post-mortem | [CKS-Equium/new-cadair](https://github.com/CKS-Equium/new-cadair) |
| [colonygame](colonygame.md) | paused | build | [CKS-Equium/ColonyGame](https://github.com/CKS-Equium/ColonyGame) |
| [hearthflix](hearthflix.md) | shipped | release | [CKS-Equium/hearthflix](https://github.com/CKS-Equium/hearthflix) |
| [team-pulse-dashboard](team-pulse-dashboard.md) | shipped | release | [CKS-Equium/team-pulse-dashboard](https://github.com/CKS-Equium/team-pulse-dashboard) |

<!-- Regenerate this table from the frontmatter of each <slug>.md (excluding _template.md). -->
