# forjador-de-skills

**A Claude Code skill for forging other well-made Claude Code skills**

## What is this?

This repository contains one skill — `forjador-de-skills` — that captures the proven pipeline for turning a pile of notes into a finished, published Claude Code skill plugin: repo anatomy, the authoring loop, distribution via a GitHub marketplace, and the naming/privacy conventions. It ships **copyable templates** so a new skill repo can be stamped out in minutes.

### Why this skill exists

Building a skill plugin "by hand" each time means re-deriving the same decisions: which files the repo needs, how marketplace auto-update actually works, where private data could leak, how to name things without colliding with the target tool's own vocabulary. This skill encodes those once.

It does **not** replace:
- **`superpowers:writing-skills`** *(required)* — the RED-GREEN-REFACTOR discipline + Claude Search Optimization for skill content.
- **`skill-creator`** *(optional helper)* — scaffolding and eval files. Not needed for the core flow (this skill ships its own templates and validates with a cheap GREEN check). ⚠️ Its optimizer/benchmark spawns **many** headless `claude -p` runs and **burns usage fast** — don't run it as a routine check.

It **adds** the packaging + distribution conventions specific to this setup (the `<app>-skills` marketplace pattern).

## The skill

| Skill | Description |
|-------|-------------|
| **forjador-de-skills** | Repo anatomy (core + optional components with triggers), the 8-step authoring pipeline, marketplace distribution + auto-update mechanics, naming/privacy conventions, and copyable templates |

## Installation

Add this marketplace in Claude Code:

```
/plugin → Marketplaces → Add Marketplace → milojarow/forjador-de-skills
```

Then install:

```
/plugin → Discover → forjador-de-skills → Install
```

## Requirements

- Claude Code with the **`superpowers`** plugin available (required — the authoring discipline).
- **`skill-creator`** is **optional** — only for its scaffolding/eval helpers. ⚠️ Its optimizer/benchmark runs many headless `claude -p` instances and burns usage fast; prefer this skill's built-in GREEN validation.
- A GitHub account, for publishing skill repos as marketplaces.

## License

MIT
