# Distributed-product QA — validating a skill that installs or runs code

The GREEN findability check ([authoring-pipeline.md](authoring-pipeline.md) step 7) proves the *docs are findable*. It does **not** prove the skill works on a machine that isn't the author's. A skill that ships an **installer**, **hooks**, a **daemon**, or **headless Agent-SDK subagents** is a distributed product — and a functional smoke on the author's box is the worst place to catch portability bugs, because that environment silently satisfies exactly the assumptions that break everywhere else.

> **"Validated on my machine" ≠ "validated as a distributed product."** A skill can pass a functional smoke, be declared ready, ship — and then break on the first fresh install with a chain of bugs the author's environment masked.

Run these checks on top of the GREEN check whenever the skill installs or runs anything.

## 1. Doc ↔ code coherence (run especially after a rename/redesign)

A functional smoke proves the code RUNS; it does NOT prove the docs tell the truth. After any rename or redesign, re-read every reference doc + every script-header comment against the **current** code: env-var names, log-file names, component names, "step N" references.

Failure shapes to hunt:
- env vars documented under an OLD name that nothing reads anymore;
- a subagent's header comment describing OLD behavior (e.g. "proposes X" when it now owns and edits X);
- a troubleshooting section describing a deleted component and its dead log lines.

Distinguish **legitimate historical mentions** ("the old X", "replaces Y", "same shape the old X used") — KEEP — from **descriptions of current behavior still using a dead name** — FIX. A blind find/replace gets this wrong; it needs judgment.

## 2. The installer installs EVERY runtime dependency

If the skill ships code whose deps don't travel in the repo (`node_modules` is gitignored, a Python venv), the installer MUST recreate them. Shipping `package.json` + lockfile but never running `npm ci` works on the author's machine (deps already present) and dies with `rc=1` / `MODULE_NOT_FOUND` on every fresh machine — worst when a **hook** launches that code automatically and it fails silently on every session-end.

"Ships the config, you bring the dep" is acceptable ONLY for a genuinely optional feature the user opts into — NEVER for something the hooks fire on their own.

**Test:** would it run on a machine where the dep was never installed by hand?

## 3. Install deps in a STABLE dir, not the per-version plugin cache

WHERE the installer puts deps matters as much as installing them. A plugin's cache lives in a **per-version** dir (`.../<version>/`). Marketplace auto-update drops a NEW version dir and does NOT re-run the installer — so deps (or a `node_modules`) living in the cache dir land in a fresh empty dir on every update, and the runtime goes silently mute (`rc=1`) until someone re-installs by hand.

- Install heavy deps ONCE in a stable home that survives updates (e.g. a venv under `~/.local/share/<skill>/...`).
- If the runtime must resolve them from the cache dir (e.g. Node ESM imports, which ignore `NODE_PATH`), have the launcher create a **symlink** from the cache's dep dir to the stable install — the symlink is recreated for free on each update; the heavy dep is installed only once.

**Rule of thumb:** deps survive updates iff they live OUTSIDE the per-version cache. (Same instinct that already puts a Python venv in a stable dir — apply it to every runtime dep, Node included.)

## 4. The self-check/status command must use the runtime's own method

If the skill ships a `status` / health command, its probe must match how the real runtime checks the same thing — otherwise it emits FALSE NEGATIVES that send the next person debugging a non-problem. Failure shape: a status script reports "daemon socket ping failed" while the daemon is healthy and serving sub-second queries, because the script pings differently from the live code path. **A status that lies is worse than no status at all.**

## 5. Write the "done" marker ONLY on success

A skill whose hooks run periodic work behind a **frequency gate** (a stamp/timestamp of the last run — "only run if >24h since last time") has a sharp edge: WHEN you write the stamp matters as much as whether you write it.

**The bug:** writing the gate stamp *unconditionally* — capturing the exit code, then stamping without checking it — means a FAILED run stamps the gate anyway. The job is then locked out for the whole window (24h, 7d, …) WITHOUT having ever run successfully. Worse, "just wait for the schedule" does not recover it — the gate is blocked by a *failure*, not a success, so it never reopens on its own until the window elapses. Classic shape: `rc=$?; echo "$now" > "$STAMP"` — stamped whether `rc` is 0 or nonzero. A first install where a runtime dep is missing (`rc=1`, see check 2) stamps the gate and locks the maintenance job out for the full window without ever running once.

**The fix** — write the marker only on success:
```
rc=$?
if (( rc == 0 )); then echo "$now" > "$STAMP"; else log "rc=$rc — not stamped, retries next run"; fi
```
A failed run leaves the gate open → it retries on the next trigger. Same discipline as a watermark/cursor that must NOT advance on failure (advance it early and you skip the unprocessed delta forever).

**General rule:** any control-flow side-effect that means "this is done, don't redo it" — gate stamps, watermarks, processed-markers, done-locks — must be written AFTER and CONDITIONAL ON success. Writing the marker before the success check turns a transient failure into a permanent (or window-long) stall, and the symptom is confusing ("it never runs") because the cause is a stamp from a run that *failed*.

## Pre-publish checklist (simulate a fresh machine)

- [ ] no hardcoded `$HOME` / username anywhere (see [naming-and-privacy.md](naming-and-privacy.md) — privacy gate)
- [ ] docs match the current code (check 1)
- [ ] installer recreates ALL deps and runs clean on a box that never had them (check 2)
- [ ] deps live in a stable dir and survive a plugin update — not the per-version cache (check 3)
- [ ] the self-check/status command agrees with reality and uses the runtime's own probe (check 4)
- [ ] every gate stamp / watermark / "done" marker is written only on `rc=0` — a failed run must retry, never lock the window (check 5)
