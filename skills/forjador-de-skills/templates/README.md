# templates/

Copyable skeletons for stamping a new skill repo. Copy these into `~/skills-dev/drafts/<app>-skills/` (dropping the `.tmpl` suffix and placing each at the path below), then replace every `{{PLACEHOLDER}}`.

> **This `README.md` is the legend — it is NOT copied into a new repo.** Only the `*.tmpl` files and `LICENSE` (listed below) get stamped.

## Placeholders

| Placeholder | Meaning |
|---|---|
| `{{REPO_NAME}}` | repo / marketplace / plugin name, e.g. `espocrm-skills` |
| `{{OWNER}}` | GitHub owner, e.g. `milojarow` |
| `{{sub-skill-name}}` | a skill under `skills/`, e.g. `espocrm` |
| `{{Sub-skill Title}}` | human title for the SKILL.md H1 |
| `{{MARKETPLACE_DESCRIPTION}}` / `{{PLUGIN_DESCRIPTION}}` | catalog blurbs (what it is — fine here) |
| `{{CATEGORY}}` | marketplace category, e.g. `productivity` |
| `{{keyword-N}}` | search keywords |
| `{{YEAR}}` | copyright year |
| `{{TRIGGERS}}` | SKILL.md `description` — WHEN-to-use ONLY, no workflow summary |

## Where each file goes

| Template | Destination |
|---|---|
| `marketplace.json.tmpl` | `.claude-plugin/marketplace.json` |
| `plugin.json.tmpl` | `.claude-plugin/plugin.json` |
| `README.md.tmpl` | `README.md` (repo root) |
| `CLAUDE.md.tmpl` | `CLAUDE.md` (repo root) |
| `LICENSE` | `LICENSE` (repo root) — fill `{{YEAR}}` and `{{OWNER}}` |
| `SKILL.md.tmpl` | `skills/<sub-skill-name>/SKILL.md` |

## Optional templates

| Template | Use when | Destination |
|---|---|---|
| `mcp.json.example.tmpl` | the skill depends on / recommends an MCP | `.mcp.json.example` (repo root) |

For which OTHER optional components a repo needs (`hooks/`, `build.sh` + `dist/`, `docs/`, `skills.png`), see [`../reference/repo-anatomy.md`](../reference/repo-anatomy.md).
