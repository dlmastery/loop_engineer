---
name: harness-prep
description: Composite meta-skill for preparing the harness — the environment a single agent runs inside — so a loop can run on top of it safely. Use when the user wants to "set up the harness", "make this repo loop-ready / agent-ready", "prepare for autonomous agents", or before standing up a loop on an unprepared repo. Orchestrates harness-bootstrap, harness-project-skills, harness-verification, and harness-guardrails. The loop-engineer skill builds the loop; this skill builds the floor it stands on.
---

# Harness Prep

Loop engineering sits one floor above the harness. The harness is the
environment one agent runs inside; the loop is the harness on a timer, spawning
helpers and feeding itself. A loop on an unprepared harness amplifies whatever
is broken — so prepare the floor before you build the loop.

Prepare the harness in this order. Each step names a component skill.

| Step | Component skill | What it guarantees |
|------|-----------------|--------------------|
| 1 | `harness-bootstrap` | The repo is reproducibly buildable/testable by a cold agent |
| 2 | `harness-project-skills` | Project intent is written down, not re-guessed each run |
| 3 | `harness-verification` | "Done" is a command that returns pass/fail, not a claim |
| 4 | `harness-guardrails` | Cost, blast radius, and permissions are bounded |

## Why order matters

- **Bootstrap before everything**: if a fresh agent can't install deps and run
  the build/tests in one documented command, nothing downstream is trustworthy.
- **Skills before the loop**: a cold agent fills any gap in your intent with a
  confident guess. Skills are that intent written on the outside, read every run.
  Without them the loop re-derives your whole project from zero every cycle.
- **Verification before automation**: you cannot let a loop decide "done" until
  "done" is a script. This is what `loop-subagents` and `/goal` check against.
- **Guardrails before unattended runs**: an unattended loop with no budget,
  broad write permissions, and no audit trail is the failure mode, not an edge
  case.

## Readiness checklist

The harness is loop-ready when **all** of these are true. Drive each to green
using the named skill:

- [ ] A cold checkout builds and tests with documented commands (`harness-bootstrap`)
- [ ] `CLAUDE.md` exists and points to the project skills (`harness-bootstrap`)
- [ ] Conventions, build steps, and "we don't do X because of Y" are captured as
      skills, not tribal knowledge (`harness-project-skills`)
- [ ] There is one command that returns a clean pass/fail for the whole repo,
      and per-area checks for tighter loops (`harness-verification`)
- [ ] A token/iteration/time budget and a kill switch exist (`harness-guardrails`)
- [ ] Agent file-write and tool permissions are scoped, not blanket
      (`harness-guardrails`)
- [ ] Secrets are in env/secret store, never in skills or state files
      (`harness-guardrails`)

When every box is checked, hand off to the `loop-engineer` skill to build the
loop itself.

## Anti-patterns this skill exists to prevent

- Scheduling a loop on a repo where `npm test` doesn't run cleanly from a fresh
  clone. The loop will "fix" phantom failures.
- Relying on the conversation to carry project conventions. The next run starts
  cold; the conventions evaporate. Write them down.
- Treating an agent's "I verified it works" as verification. It isn't until a
  command says so.
- Giving the loop unbounded budget "just to see." That's how token-rich becomes
  token-poor.
