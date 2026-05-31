# Naming & privacy conventions

## Naming

- **Repos:** `<app>-skills`, where `<app>` is the official product-name token (`espocrm` → `espocrm-skills`; "Meta Ads", plural → `meta-ads-skills`). Verify the official spelling — don't guess singular/plural.
- **Multi-tool pipelines** break `<app>-skills` (no single app). Name the repo by **function** and surface the tools in the `description` + in tool-named sub-skills. Example: an AI-video stack (HyperFrames + ElevenLabs + kie.ai) → `ai-video-skills` with `skills/hyperframes-composition`, `skills/elevenlabs-voice`, `skills/kieai-assets`.
- **Never name coined things after the target tool's own vocabulary.** Auth paths, modes, roles you invent must not collide with terms the target platform already defines — it confuses and can invert the real mapping. (E.g. don't label EspoCRM's two auth paths `admin`/`super-admin` — EspoCRM HAS those user types. Name by what authenticates: `api-user path` / `admin path`.)
- **Check for collisions across already-installed skills.** (Don't name a video skill `hyperframes-*` when HeyGen's official `hyperframes` skills are installed.)
- **Reference files:** kebab-case (`api-endpoints.md`, `common-errors.md`).
- **`name` frontmatter / sub-skill dirs:** letters, numbers, hyphens only.

## Privacy gate (HARD — before anything touches a repo)

A skill repo may be public. The **scrub is the safety net, not `.gitignore`.** Reference files and templates must be generic:

- No client/business names, no real people's names.
- No real record / team / user IDs (e.g. EspoCRM 17-char hex).
- No instance URLs, container names, server hostnames.
- No credentials, API keys, or secrets-file paths.
- No business-model specifics (pricing, internal arrangements).
- **No hardcoded `$HOME` / username** in code, agents, scripts, or docs — a literal `/home/<user>`, a home-derived slug, or a personal name baked into an agent file. Two harms at once: it leaks a personal name into a public repo, AND it breaks on any machine with a different `$HOME`. Use `$HOME` / runtime detection (e.g. a slug via `echo "$HOME" | tr / -`), never a literal. Pre-publish grep across `*.sh`/`*.mjs`/`*.md`/`*.json`:

  ```bash
  grep -rn '/home/[a-z]' <repo>          # + grep your own username
  ```

  (The portability side of this check also lives in the distributed-product QA — see [distributed-product-qa.md](distributed-product-qa.md).)

Scrub the **source** before the first commit, so the repo's git history never holds the real data. Verify against **history**, not just the working tree:

```bash
git -C <repo> log --all -S '<a-real-value>'    # expect empty (never committed)
grep -rniE '<signature-pattern>' <repo>        # expect CLEAN
```

If a real value ever entered a commit, history must be rewritten (or the repo recreated) before publishing — a deleted-but-committed secret is still public.

## Commit & repo conventions

- Atomic, imperative, English commit subjects. Bump the version in both manifests in the same commit as the change.
- End commit messages with the `Co-Authored-By` trailer.
- `CLAUDE.md` **is committed** in skill repos (public repo guidance). `.claude/memory/` is **never** committed, anywhere.
- LICENSE: MIT unless the operator says otherwise.
