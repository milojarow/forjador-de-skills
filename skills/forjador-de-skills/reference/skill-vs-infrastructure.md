# Skill vs infrastructure — classify before you package

Before packaging something as a skill, classify what it actually is. The two kinds want opposite `description` (CSO) treatment, and getting it wrong taxes every prompt in the domain.

- **Knowledge-to-act** — APIs, gotchas, domain conventions where the **model** performs the task. It needs the skill in context *while it works*. → a classic skill; CSO triggers whenever the domain work happens.
- **Infrastructure-that-runs** — hooks, daemons, headless subagents that fire on harness events. The machinery runs on its own; the model never reads a manual to make it work. → the plugin is still a good **vehicle** (a `hooks.json` + scripts + agents + installer distribute and auto-update through the marketplace), but its skill component must carry a **maintenance-only CSO**.

## The maintenance-only CSO

For an infrastructure plugin, the `description` must trigger **only when working ON the system** — installing, migrating, tuning, debugging, extending — and must **explicitly exclude** the routine operation the infrastructure already handles by itself. State what it does **not** trigger (the routine operation) with the same clarity as what it does.

> The skill is the manual for the machinery; the machinery runs without the model reading the manual.

## Anti-pattern symptom

The skill loads on every domain prompt even though the hooks/agents do the work autonomously — spending tokens and attention to document machinery that runs without the model knowing how. The added risk: a model holding the manual tends to do **by hand** what the infrastructure already does — duplicating writes and stepping on the pipeline.

## Validated case

An infrastructure plugin's CSO was narrowed from a broad "use when the user says *save this / remember / recall*" (which fired on every memory-domain prompt) to maintenance-only — **without touching its hooks, agents, or scripts**. The infrastructure kept running identically; the per-prompt context cost disappeared.
