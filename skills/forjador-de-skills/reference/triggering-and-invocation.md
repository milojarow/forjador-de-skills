# Does the skill actually fire? — measure it, then make invocation deterministic

A **router skill** — the one that says *"invoke X in phase 1, Y in phase 2"* — does **not**
fire just because its `description` carries the right words. A polished description decides
whether the model can *find* the skill once it goes looking; whether it looks at all is a
separate problem, and it is solved **outside** the skill.

Treat "does it trigger?" as a claim to be measured, not a property to be assumed.

## Measuring it — cheap, twice, no loops

Run a **fresh headless session** in a directory whose skill listing matches what the user
will have, with a realistic prompt that does **not** name the skill:

```bash
CLAUDE_HEADLESS=1 claude -p "<realistic prompt, WITHOUT naming the skill>" \
  --permission-mode plan --max-turns 2 --output-format stream-json --verbose > run-A.jsonl

jq -r 'select(.type=="assistant") | .message.content[]? | select(.type=="tool_use")
       | "\(.name)\t\(.input.skill // "")"' run-A.jsonl
```

Then the **positive control** (run B): the same prompt with *"use the `<name>` skill"* in
front. Without B, a silent run A proves nothing — it could be the prompt, the harness, or a
broken invocation path.

**Reading the result:** if B fires and A does not, the description is not enough. The listing
competes with a couple of hundred entries *and* with always-on imperatives (e.g. a
"brainstorm first" rule). Measured on a real router: run A produced brainstorming plus three
`ls` calls and never the skill; run B loaded the skill first.

**Cost and discipline:** roughly a dollar or so per 3-turn run against a large listing. Two
runs are enough. This is exactly the *deliberate, cost-guarded* use of a headless benchmark
that [authoring-pipeline.md](authoring-pipeline.md) step 7 reserves for when triggering
itself is the open question — never a routine "does this work" check, and never in a loop.

**Two traps in the harness itself:**

- A hook that blocks `claude -p` from Bash is **correct** (fork-bomb guard) — do not delete
  it. Its own refusal text carries the escape hatch for a single run: flip it to
  `enabled: false`, run, and re-enable from a `trap` so an aborted run cannot leave the guard
  down.
- A headless run that reaches `SessionEnd` can launch background agents. `CLAUDE_HEADLESS=1`
  stops them **only if those scripts check for it** — grep them before you run, don't assume.

## Making it deterministic — cheapest layer first

1. **An always-on imperative** in the user's or the project's `CLAUDE.md`: *"for `<task
   type>`, invoke `<plugin>:<skill>` first."* Always-on imperatives **do** fire — the same
   test proves it, since the harness's own brainstorming rule was obeyed in run A while the
   new skill never loaded.
2. **Name the skill in the prompt** — four words, zero infrastructure, measured.

A `UserPromptSubmit` hook that regex-matches the prompt and injects the instruction also
works, but it is machinery with its own failure modes: reach for it only after the `CLAUDE.md`
imperative has been *measured* insufficient.

## Corollary for the author

Write the description for **findability**, then get the invocation from somewhere else. This
is the second half of the router disappointment: a router does not shrink the skill listing
either (see [gating-and-token-cost.md](gating-and-token-cost.md)) — so it buys neither
cheapness nor reliable firing. What it buys is a documented order of work, once something
actually invokes it.
