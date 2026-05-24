# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

This is the **forjador-de-skills** repository — a meta-skill that teaches Claude Code how to forge other well-made skill plugins.

**Repository**: https://github.com/milojarow/forjador-de-skills

**Purpose**: capture the proven pipeline for building, packaging, and distributing a Claude Code skill plugin — repo anatomy, the authoring loop, marketplace distribution, and naming/privacy conventions — plus copyable templates. A thin layer over `superpowers:writing-skills` and `skill-creator`.

## Repository Structure

```
forjador-de-skills/
├── .claude-plugin/          # Claude Code plugin configuration
├── CLAUDE.md                # This file
├── README.md                # Project overview
├── LICENSE                  # MIT License
├── evaluations/             # Test scenarios for the skill
└── skills/
    └── forjador-de-skills/  # The meta-skill
        ├── SKILL.md         # Entry point: when to use + the pipeline
        ├── reference/       # repo-anatomy, authoring-pipeline, distribution, naming-and-privacy
        └── templates/       # Copyable skeletons for stamping a new skill repo
```

## The skill

### forjador-de-skills
The methodology for forging a Milo-style skill plugin. Covers repo anatomy (core components always; `hooks/`, `.mcp.json.example`, `build.sh`+`dist/`, `docs/` as optional-with-triggers), the 8-step authoring pipeline (scrub → scaffold → migrate → polish → evals → git → validate → refactor), distribution via a GitHub marketplace (auto-update fires at startup only), and naming/privacy conventions. Leans on `superpowers:writing-skills` and `skill-creator` — does not duplicate them.

## Skill Activation

Activates when the task is building, packaging, or publishing a new Claude Code skill plugin in this setup — scaffolding a repo, deciding which files it needs, wiring the marketplace, or naming a new skill.

## Updating this skill

After any cycle that teaches a new packaging/distribution lesson. Keep entries generic — patterns and examples, never client data. The git log of this repo is the diary.
