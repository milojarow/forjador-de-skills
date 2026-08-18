# Gating a plugin per repo — and measuring what it costs to leave it on

A published plugin is not free. Every enabled plugin pays rent in the skill listing of
**every** session, whether or not its skills ever fire. This file covers the two halves:
how to scope a plugin to the repos where it belongs, and how to price it before installing.

## Gating: `--scope`, not a router skill

Recurring request: *"I want these N skills to exist only when I work on this kind of
project."*

The answer is **not** a router skill. A router does not shrink the listing — the child
skills are still listed, so the always-on cost is unchanged. The answer is the CLI:

```bash
claude plugin disable <plugin> --scope user      # off everywhere
cd <repo>
claude plugin enable  <plugin> --scope project   # on only here
```

`--scope` takes `user | project | local` and follows the documented settings precedence
(**Local > Project > User**). The project scope writes the `enabledPlugins` key into
`<repo>/.claude/settings.json`, which is committable — so the gating travels with the repo.

**Installing and enabling are separate facts.** The install lives in
`~/.claude/plugins/installed_plugins.json` (with `installPath` and version); the enablement
lives in `enabledPlugins` in settings. That is why a plugin can be installed and switched
off at the same time — and why "it's installed" answers nothing about whether it costs you
anything.

## Price it BEFORE installing

```bash
claude plugin details <plugin>
```

Returns the component inventory (skills, agents, hooks, MCP, LSP) and a **projected token
cost**, split in two:

- **always-on** — what every session pays merely because the plugin is enabled (the listing).
- **on-invoke** — what you pay each time a skill actually fires.

Measured on a real 15-skill pack: **~3,809 tokens always-on ≈ ~250 tokens per skill**. Rule
of thumb for budgeting a new pack: `number of skills × ~250 tokens`. Hooks come back marked
*harness-only — no model context cost*.

## Why always-on is the number that matters

The skill listing has a budget: `skillListingBudgetFraction` (default **1%** of the context
window; raisable, e.g. `0.02`). When the listing exceeds that budget, Claude Code **drops
the descriptions of the least-used skills** and keeps only their names.

The counter-intuitive, expensive consequence: **a large, rarely-used pack degrades the
triggering of the skills you DO use**, because it eats their share of the budget. The damage
is not the pack's own cost — it is the collateral.

Details that change the arithmetic:

- **Names are always listed**, no matter what. Only descriptions get trimmed.
- Each entry (description + when-to-use) is capped at **1,536 chars**
  (`skillListingMaxDescChars`). That is why the primary use case goes **FIRST** in the
  description — the tail is what gets cut.
- **`/doctor`** estimates the listing cost against the budget and names the biggest
  contributors. That is how you learn you are already over it, instead of assuming.

## The finer lever — and its hard limit

`skillOverrides` in settings gives four states per skill:

`on` · `name-only` · `user-invocable-only` · `off`

`name-only` is exactly the tool for freeing budget without uninstalling anything; the
`/skills` menu writes it for you with Space + Enter into `.claude/settings.local.json`.

**But the documentation is explicit: `skillOverrides` does NOT apply to plugin skills.** For
plugins the only controls are `/plugin` and `--scope`.

That is the trade to make **at packaging time, not after**: shipping something as a plugin
buys per-repo gating and gives up per-skill granularity. If a pack contains one skill that
should be permanently `name-only` and four that should stay hot, packaging them together
forecloses that.

## Packaging consequence

Two rules fall out for a forjador deciding how to cut a pack:

1. **Split by activation moment, not by subject matter.** Skills that are only ever needed
   inside one kind of repo belong in their own plugin, so they can be `--scope project`
   there and off everywhere else.
2. **A pack's size is a cost imposed on unrelated sessions.** Before adding the Nth skill to
   an existing plugin, ask whether every session that enables the plugin should pay ~250
   tokens for it — and run `/doctor` afterwards rather than assuming the budget absorbed it.
