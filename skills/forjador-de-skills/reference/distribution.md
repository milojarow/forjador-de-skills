# Distribution — publishing and updates via the marketplace

## The marketplace IS the repo

There is no central store. A GitHub repo with a `.claude-plugin/marketplace.json` *is* a marketplace — it advertises itself, and Claude Code points directly at the repo.

- `marketplace.json` — the catalog (which plugins the repo offers).
- `plugin.json` — the plugin's own manifest.

## Publish (only on the operator's "ready" — the install gate)

```bash
gh repo create <owner>/<repo> --public --source <repo-dir> --remote origin --push --description "..."
```

Then the operator, in their Claude Code session:

```
/plugin marketplace add <owner>/<repo>
```

and installs the plugin via `/plugin → Discover`. The marketplace is recorded in `~/.claude/plugins/known_marketplaces.json`; the install in `~/.claude/plugins/installed_plugins.json`.

## Shipping an update

1. Edit the repo.
2. **Bump the version in BOTH manifests** — `marketplace.json` (`plugins[].version`) AND `plugin.json` (`version`). Keep them consistent.
3. `acp` (add + commit + push).

## How the update reaches the operator (the #1 confusion)

Auto-update fires at **Claude Code STARTUP only** — not on a timer, not on demand. After `acp`, the **running** session does NOT update live.

- **`/reload-plugins` does NOT fetch from the remote.** It only reloads locally-cached plugin definitions; the marketplace clone stays on its current commit.
- To pull a new version **now**, either:
  - **Restart Claude Code** — startup pulls the marketplace clone AND reinstalls the plugin (one move, does both); or
  - **`/plugin marketplace update <name>`** then **reinstall** the plugin — these are SEPARATE steps; updating the marketplace clone does NOT auto-reinstall the cached plugin.

## `autoUpdate` gotcha

`/plugin marketplace add` does NOT necessarily enable `autoUpdate` for the marketplace. Verify `known_marketplaces.json` shows `"autoUpdate": true` for the repo; if not, enable it — otherwise the `acp` → startup-update loop never fires.

The global `autoUpdates` setting of Claude Code itself is **separate** and does NOT gate per-marketplace plugin auto-update (verified: a plugin updated on restart even with the app's own auto-update off).

## No build artifacts by default

Marketplace-from-repo serves straight from the repo. `build.sh` / `dist/` (versioned zips) exist only for standalone-zip distribution channels (e.g. Claude.ai uploads) — skip them unless you need that channel.
