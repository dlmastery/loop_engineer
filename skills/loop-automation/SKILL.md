---
name: loop-automation
description: The heartbeat of a loop — schedule discovery and triage so the loop runs on a cadence instead of you re-running it. Use when setting up recurring agent runs, "run this every morning/N minutes", cron tasks, in-session /loop or /goal, lifecycle hooks, or GitHub Actions for autonomous agents. Covers choosing in-session vs out-of-session triggers, cadence, and routing findings to a triage inbox.
---

# Loop Automation (the heartbeat)

Automations are what make a loop an actual loop and not one run you did once.
This skill picks the trigger and cadence, and makes sure findings come to you
instead of you going to check.

## Pick the trigger

| You want… | Use | Notes |
|-----------|-----|-------|
| Re-run a prompt/command on an interval, in this session | `/loop <interval> <prompt-or-/command>` | Omit interval to let the model self-pace |
| Keep going until a *verifiable* condition holds | `/goal "<stop condition>"` | A separate small model grades "done" each turn — maker/checker on the stop condition itself |
| A recurring cloud agent on a cron schedule | `/schedule` (routines) | Survives closing the laptop |
| Fire shell commands at points in the agent lifecycle | hooks (`settings.json`) | e.g. format on edit, gate on stop |
| Keep running in CI after you log off | GitHub Actions | Durable, reviewable, logs retained |

Rule of thumb: **`/loop` for cadence, `/goal` for a finish line.** Prefer
`/goal` whenever there's a checkable stop condition — it bakes in the verifier.

### In-session vs out-of-session
- **In-session** (`/loop`, `/goal`): fast to set up, dies with the session, you
  watch it. Good for a focused push you're supervising.
- **Out-of-session** (cron/`/schedule`, Actions, hooks): runs when you're gone,
  must be fully autonomous and fully guarded. Good for daily triage.

## Cadence

- Match cadence to how fast the input actually changes. Nightly for CI/issue
  triage. Per-commit (hook) for fast feedback. Hourly is rarely right.
- Decide local checkout vs background worktree per run. Discovery that only
  reads → local is fine. Anything that writes → background worktree (see
  `loop-worktrees`).
- Runs that find nothing should archive themselves. Only runs with findings
  should demand attention.

## Call a skill, not a wall of text

An automation's prompt should invoke a skill (e.g. a `triage` skill), not paste
a giant instruction block into a schedule nobody will maintain. Keep the
recurring instruction in a versioned skill; the automation just fires it.

```
# good: maintainable
/schedule "every weekday 7am: run the triage skill on this repo"

# bad: a 600-word prompt frozen into a cron entry
```

A ready-to-adapt triage skill is in this repo: `templates/triage-skill/SKILL.md`.

## Route the findings

Every run must end by writing somewhere durable:
- findings the loop will act on → the state file (see `loop-memory`),
- findings needing a human → a **triage inbox** (a markdown section, a Linear
  column, a label). Never let a finding evaporate when the run ends.

## Before you schedule it

1. Run the automation's prompt **once, by hand, attended.**
2. Give the first scheduled runs a tight budget and a kill switch
   (`harness-guardrails`).
3. Confirm a "nothing found" run is cheap and quiet.

Only then turn on the cadence. An unattended automation is also an unattended
mistake generator — earn the cadence with one clean supervised run.
