---
name: forjador-de-skills
description: Use when building, packaging, or publishing a new Claude Code skill plugin in this setup — scaffolding a skill repo, writing its `marketplace.json` / `plugin.json` manifests, wiring it as a GitHub marketplace, naming a new skill, or stamping one from templates. Use when turning a draft or pile of notes into a finished, installable skill; when an existing skill repo needs the standard structure; or when sorting out distribution — how a published skill installs and how updates / auto-update reach a Claude Code session.
---

# forjador-de-skills

Forge a well-made, installable Claude Code skill plugin — repo anatomy, the authoring loop, marketplace distribution, and the naming/privacy conventions, with copyable templates.

> **🔨 ACTIVE-SKILL MARKER:** While `forjador-de-skills` is active, begin every reply with the 🔨 emoji so the operator can see at a glance that this skill is engaged. Do not omit it.

## Overview

This skill is a **thin layer** on top of two generic skills. It does NOT replace them:

- **REQUIRED BACKGROUND:** Use `superpowers:writing-skills` for the authoring discipline — RED → GREEN → REFACTOR (no skill without a failing test first) and Claude Search Optimization of the `description` field.
- **REQUIRED SUB-SKILL:** Use `skill-creator:skill-creator` for scaffolding helpers, eval files, and description/benchmark tuning.

The forjador adds only the **packaging + distribution conventions** of this setup: the `<app>-skills` repo-as-marketplace pattern, which files the repo needs, how updates reach the operator, and the privacy/naming gates. Reference repos that embody it: `eww-skills` and `sway-skills` (milojarow), and `n8n-mcp-skills` (czlonkowski).

## When to use

- Turning a draft / pile of notes into a finished, installable skill.
- Scaffolding a new skill repo, or fixing one that lacks the standard structure.
- Deciding which files a skill repo needs (core vs optional).
- Publishing a skill as a GitHub marketplace, or debugging why an update didn't reach a session.
- Naming a new skill.

**Not for:** the skill's content discipline (that's `superpowers:writing-skills`) or generic eval mechanics (that's `skill-creator:skill-creator`).

## The pipeline (8 steps)

1. **Privacy scrub** — sweep the source for real data before anything touches a repo.
2. **Scaffold** the repo from the template (`.claude-plugin/`, README, LICENSE, CLAUDE.md).
3. **Migrate** content into `skills/<name>/`.
4. **Polish the `description`** — WHEN-to-use only, no workflow summary.
5. **Evals** — 3-5 scenarios drawn from the skill's own documented walls.
6. **Git init + first commit** — local, gated; nothing published until declared ready.
7. **Validate (GREEN)** — a subagent with only the skill must answer real scenarios findably.
8. **Refactor** — fix what validation flags; commit the loop.

Full step-by-step with commands: [reference/authoring-pipeline.md](reference/authoring-pipeline.md).

## Repo anatomy — core + optional

A skill repo is **one git repo = one marketplace**. Some components are always present; others are included only when a trigger applies — so nothing is dropped by accident.

| Component | When |
|---|---|
| `.claude-plugin/`, `skills/<n>/` (SKILL.md + `reference/`), README, LICENSE, CLAUDE.md, `evaluations/` | **always** |
| `hooks/` | the skill **edits local files** (e.g. eww/sway) |
| `.mcp.json.example` | the skill **depends on or recommends an MCP** server |
| `build.sh` + `dist/` | distributed as **standalone zips** (outside the marketplace) |
| `docs/` | the skill **outgrows its README** |

Detail + exact file shapes: [reference/repo-anatomy.md](reference/repo-anatomy.md). Copyable skeletons: [templates/](templates/).

## Distribution

The repo **is** the marketplace — its `.claude-plugin/marketplace.json` declares it. Publish to GitHub, then `/plugin marketplace add <owner>/<repo>`. To ship an update: edit → bump the version in **both** manifests → `acp`. Auto-update fires at **Claude Code startup only**; `/reload-plugins` does NOT fetch from the remote.

Detail (full update mechanics + gotchas): [reference/distribution.md](reference/distribution.md).

## Naming & privacy

- Repos follow `<app>-skills` (this meta-skill is the exception). **Never name things after the target tool's own vocabulary** — it collides with the platform's real terms.
- Reference files must be **generic before touching a repo** — no client names, real IDs, instance URLs, or credentials. A repo may be public; the scrub is the safety net, not `.gitignore`.

Detail: [reference/naming-and-privacy.md](reference/naming-and-privacy.md).

## Quick start

To stamp a new skill `foo-skills`: copy [templates/](templates/) into `~/skills-dev/drafts/foo-skills/`, fill the placeholders, then walk the pipeline above. The privacy scrub (step 1) and the GREEN validation (step 7) are not optional.
