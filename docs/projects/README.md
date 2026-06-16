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
| [team-pulse-dashboard](team-pulse-dashboard.md) | active (v1.1) | build | [CKS-Equium/team-pulse-dashboard](https://github.com/CKS-Equium/team-pulse-dashboard) |
| [hearthflix](hearthflix.md) | shipped | release | [CKS-Equium/hearthflix](https://github.com/CKS-Equium/hearthflix) |
| [colonygame](colonygame.md) | parked (DEV_PLAN_2 mid-flight) | build | [CKS-Equium/ColonyGame](https://github.com/CKS-Equium/ColonyGame) |

<!-- Regenerate this table from the frontmatter of each <slug>.md (excluding _template.md). -->
