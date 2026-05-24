# Repo anatomy — core + optional components

One skill repo = **one git repo = one marketplace**. The marketplace serves one or more plugins; a plugin bundles one or more sub-skills.

## The skeleton

```
<app>-skills/
├── .claude-plugin/
│   ├── marketplace.json     # the catalog: declares the repo as a marketplace
│   └── plugin.json          # the plugin manifest
├── skills/
│   └── <sub-skill>/
│       ├── SKILL.md          # frontmatter + sectioned body, LEAN
│       └── reference/        # depth lives here (progressive disclosure)
│           └── *.md
├── evaluations/
│   └── <sub-skill>/eval-*.json
├── README.md
├── LICENSE
└── CLAUDE.md
```

## Core (always present)

- **`.claude-plugin/marketplace.json`** — declares the repo as a marketplace; `plugins[]` lists what it offers (name, `source: "./"`, version, keywords, repository, license).
- **`.claude-plugin/plugin.json`** — the plugin manifest (name, version, description, author, license, keywords, repository, homepage).
- **`skills/<sub-skill>/SKILL.md`** — the lean entry point. Frontmatter `name` + `description`; body in clear `##` sections that cross-link the reference files.
- **`skills/<sub-skill>/reference/*.md`** — the depth. Kebab-case filenames.
- **`evaluations/<sub-skill>/eval-*.json`** — 3-5 test scenarios (see [authoring-pipeline.md](authoring-pipeline.md)).
- **`README.md`** — human-facing: "What is this / Why / table of skills / install / requirements / License". No badges.
- **`LICENSE`** — MIT, "Copyright (c) <year> <owner>".
- **`CLAUDE.md`** — repo guidance (overview, structure, the skill(s), activation, update note). **Committed** — in a skill repo CLAUDE.md is a published doc, NOT operator context, so the usual "gitignore CLAUDE.md" rule does NOT apply here.

## Optional (include only when the trigger applies)

| Component | Include when | Note |
|---|---|---|
| `hooks/` | the skill **edits local files** | `hooks.json` (PreToolUse: `Read\|Edit\|Write\|Bash`) + a `pretooluse-inject.py` that reads each SKILL.md's `metadata.{pathPatterns,bashPatterns}` and injects the skill on a match. Pointless for API/MCP-only skills — the matcher never sees MCP calls. Requires a `metadata:` block in the frontmatter. |
| `.mcp.json.example` | the skill **depends on or recommends an MCP** | A placeholder config showing the server entry + required env vars; referenced from the README. |
| `build.sh` + `dist/` | distributed as **standalone zips** | Versioned `.zip` per skill, for non-marketplace channels (e.g. Claude.ai uploads). Marketplace-from-repo does NOT need them. |
| `docs/` | the skill **outgrows its README** | Extended human docs (INSTALLATION/USAGE/DEVELOPMENT). |
| `skills.png` | want a marketplace **banner** | Cosmetic image for the listing. |
| `.gitignore` | there are **build artifacts** to ignore | Only if you add `dist/`. Otherwise omit (eww/sway/espocrm have none). |

**Rule:** the anatomy is core + optional-with-triggers, **never a fixed shape**. For each new skill, walk the optional list and consciously include or skip each one, with a reason — so nothing is omitted silently.

### Implementing `hooks/` (local-file-editing skills only)

The hook is generic and data-driven — it reads each SKILL.md's frontmatter, so only the per-skill patterns change. To add it:

1. Add a `metadata` block to the sub-skill's frontmatter:
   ```yaml
   ---
   name: <sub-skill-name>
   description: ...
   metadata:
     priority: 7
     pathPatterns: ["**/eww/**/*.scss", "**/*.yuck"]
     bashPatterns: ["eww\\s+(reload|open|inspector)"]
   ---
   ```
2. Copy the proven `hooks/` from a skill that already ships it (e.g. milojarow's `eww-skills/hooks/` — `hooks.json` + `pretooluse-inject.py`). The script is generic; **don't rewrite it**, only the per-skill `metadata` patterns differ.

`hooks.json` registers a PreToolUse hook on `Read|Edit|Write|Bash` that runs `pretooluse-inject.py`; the script matches `file_path` against `pathPatterns` and bash commands against `bashPatterns`, then injects the matching skill (deduped per session). It never sees MCP tool calls — which is why API/MCP-only skills skip `hooks/` entirely.

## Sub-skill count

One repo can hold a single sub-skill (`espocrm-skills/skills/espocrm`) or many (`eww-skills` has 6, `sway-skills` has 11). Split into several when the domain has distinct concerns with their own triggers; keep one when it's cohesive.
