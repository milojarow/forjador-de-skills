# Authoring pipeline — from draft to published skill

The proven order. Steps 1 and 7 are not optional.

**REQUIRED BACKGROUND:** the *content* discipline — RED → GREEN → REFACTOR, the description rules — belongs to `superpowers:writing-skills`. This file is the *packaging* loop that wraps it.

## 1. Privacy scrub (FIRST — before anything touches a repo)

Sweep the source for real data. Working signature grep (adapt to the domain):

```bash
grep -rniE 'company-names|vps/host-names|real-phone-fragments|[a-f0-9]{17,}|https?://[a-z0-9]|@[a-z0-9.-]+\.[a-z]{2,}|secrets-paths' <draft-dir>
```

Replace real values with obvious placeholders (`+52 555 123 4567`, `<your-instance-url>`, `${API_KEY}`). Scrub the **source** before copying, so the repo's git history never contains the real data. Full gate: [naming-and-privacy.md](naming-and-privacy.md).

## 2. Scaffold the repo

Copy [`../templates/`](../templates/) into `~/skills-dev/drafts/<app>-skills/` and fill the placeholders: the two `.claude-plugin/` manifests (v0.1.0), README, LICENSE, CLAUDE.md. Layout: [repo-anatomy.md](repo-anatomy.md).

## 3. Migrate content

Copy the (scrubbed) draft into `skills/<sub-skill>/` — `SKILL.md` + `reference/`. Keep the original draft until the migration is confirmed.

## 4. Polish the SKILL.md description

`description` = WHEN-to-use triggers ONLY. Strip any "Covers… / does…" tail — a workflow summary makes Claude follow the description and skip the body. **Cross-link every major reference topic from the SKILL.md body** (not just the description) — a body-skimming agent must be able to find each reference file.

## 5. Evals

3-5 JSON scenarios drawn from the skill's OWN documented walls. Schema:

```json
{
  "id": "<skill>-001",
  "skills": ["<sub-skill>"],
  "query": "a realistic user question",
  "expected_behavior": ["activate the skill", "..."],
  "expected_content": ["keyword", "reference-file.md"],
  "priority": "high",
  "notes": "what this guards against"
}
```

Descriptive filenames: `eval-001-<topic>.json`.

**Two eval genres — don't confuse them (a repo needs the first):**
- **`eval-NNN-<topic>.json` — the schema above — STANDARD and required.** GREEN scenarios with `expected_behavior[]` that grade findability + routing into the right reference file. Cheap: graded by step 7's single in-session subagent. Ship 3-5 per sub-skill.
- **`trigger-evals.json` — OPTIONAL, cost-guarded.** A flat array of `{query, should_trigger}` booleans — fixtures for the headless trigger-benchmark (`claude -p` × hundreds → burns usage; see step 7's cost guard). It measures auto-trigger recall/precision, a *different* axis. It is **NOT a substitute**: a repo shipping only `trigger-evals.json` has zero GREEN coverage. Prefer the scenario evals; add the trigger set only when *triggering itself* is the open question.

## 6. Git init + first commit (LOCAL, gated)

```bash
git -C <repo> init
git -C <repo> add -A

cat > "$SCRATCH/msg.txt" <<'EOF'          # quoted delimiter = zero expansion
Scaffold <app>-skills plugin v0.1.0

<body — backticks, $vars and "quotes" are all safe in here>

Co-Authored-By: <model> <noreply@anthropic.com>
EOF
git -C <repo> commit -F "$SCRATCH/msg.txt"
```

No remote, no push, no `marketplace add` until the operator declares it ready (the **install gate**). `v0.1.0` = pre-release.

> **⚠️ Never pass a non-trivial commit message with `-m`.** The agent's shell is **zsh**, and
> backticks inside a `-m` string are **command substitution**: zsh tries to execute the quoted
> identifier, prints `command not found`, and the commit **still succeeds with exit 0** — with
> the word replaced by the failed command's empty output.
>
> ```text
> written:  `explicit` was the catch-all
> stored:    was the catch-all
> ```
>
> The failure is silent where it counts: the hash exists, the zsh error scrolls away in the
> output, and nobody re-reads a message after committing. Same family as the other zsh
> surprises (no word-splitting on unquoted expansion).
>
> **Check whether it already happened:** `git log -1 --format=%B` and look for the backticks.
> If they vanished together with the word between them, this is why. **Fix:**
> `git commit --amend -F <file>`.
>
> The heredoc with a **quoted** delimiter (`<<'EOF'`) is the safe pattern for *any* long text
> carrying metacharacters — commit messages, eval JSON, `description` drafts — not just commits.

## 7. Validate (GREEN — the real test)

Dispatch a subagent that has ONLY this skill: give it the file paths + the eval QUERIES (NOT the expected answers). Make it read `SKILL.md` first and drill into reference files as needed. Grade its answers against `expected_behavior`. This catches findability gaps and dangling cross-references that a self-review misses. Finding a gap is the win. **One subagent — cheap.**

> **⚠️ Don't reach for the headless benchmark by default.** `skill-creator`'s description-optimizer and any `claude -p` trigger-benchmark spawn **many** background model instances and **burn usage fast** (and the optimizer is broken on recent Claude Code). GREEN above is the default. Use a headless benchmark only deliberately — when a skill's *auto-triggering* recall/precision is the specific open question — never as a routine "does this work" check. The two-run A/B recipe for that case (and how to make invocation deterministic instead): [triggering-and-invocation.md](triggering-and-invocation.md).

## 8. Refactor

Fix what validation flags; re-verify; commit the fix **separately** so the git history shows the RED → GREEN → REFACTOR loop.

## Then: publish (only on the operator's "ready")

See [distribution.md](distribution.md).
