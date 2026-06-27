---
name: loop-engineer
description: Composite meta-skill for designing a self-prompting agent loop instead of prompting the agent turn-by-turn. Use when the user wants to "build a loop", "design a loop", "set up an autonomous/recurring agent workflow", "stop prompting and start looping", or automate discovery → fix → review → record → repeat. Orchestrates the five building blocks (automation, worktrees, memory, subagents, connectors) plus a durable state file, and refuses to ship a loop that has no verifier or no cost ceiling.
---

# Loop Engineer

You are designing the system that prompts the agent, not prompting the agent
yourself. A loop is a recursive goal: define a purpose and let the system
iterate — find work, hand it out, check it, write down what's done, decide the
next thing — until a verifiable stop condition holds.

This meta-skill orchestrates five component skills plus one memory discipline.
Use them; do not reinvent them inline.

| Block | Component skill | One-line job |
|-------|-----------------|--------------|
| Heartbeat | `loop-automation` | Schedule discovery/triage so the loop fires without you |
| Isolation | `loop-worktrees` | Keep parallel agents from clobbering each other |
| Spine | `loop-memory` | Durable on-disk state: done / in-flight / next |
| Maker/checker | `loop-subagents` | One agent drafts, a *different* one verifies |
| Reach | `loop-connectors` | Let the loop act in real tools (issues, CI, Slack) via MCP |

The harness the loop runs inside is a separate concern — see the
`harness-prep` skill before the first run if the repo isn't loop-ready.

## The non-negotiables (check these first)

A loop that runs unattended is also a loop making mistakes unattended. Before
designing anything, get explicit answers. Refuse to finish a loop that lacks
either of the first two:

1. **Verifier ≠ maker.** What independent check decides "done"? Tests passing,
   lint clean, a second model reviewing against a spec. If the only judge is the
   agent that wrote the code, the loop's "done" means nothing. (`loop-subagents`)
2. **Cost ceiling.** What stops a runaway? A max iterations / max-tokens /
   max-wallclock budget per run, and a kill switch. Token usage varies wildly;
   a loop with no ceiling is a bill with no ceiling. (`loop-guardrails` in
   `harness-prep`)
3. **Escape hatch.** Where does work the loop *can't* handle go? A triage inbox
   or a "needs human" column — never silently dropped.
4. **Review surface.** How will a human read what shipped? Your review bandwidth
   is the real ceiling on how many loops you can run, not the tooling.

If the user can't answer 1 or 2, stop and design those before writing any loop.

## Design procedure

Work top to bottom. Each step names the component skill to invoke.

### 1. Name the recursive goal
Write the loop's purpose as a verifiable stopping condition, not a vibe.
- Bad: "keep the codebase healthy."
- Good: "for each failing test in `test/`, open a worktree, draft a fix, and a
  reviewer subagent confirms the test passes and lint is clean before opening a
  PR; unfixable cases go to the triage inbox."

### 2. Choose the trigger → `loop-automation`
In-session (`/loop` on a cadence, `/goal` until a condition holds) vs.
out-of-session (cron task, GitHub Actions, lifecycle hooks). Decide cadence and
whether each run is local or on a background worktree.

### 3. Lay down the memory → `loop-memory`
Create the state file *before* the first run. The model forgets everything
between runs; the repo doesn't. This file is the spine — it records what was
tried, what passed, what's still open, so tomorrow's run resumes today's.

### 4. Isolate parallel work → `loop-worktrees`
If the loop fans out to more than one agent at once, each gets its own worktree
so edits can't collide. Skip if strictly serial.

### 5. Split maker from checker → `loop-subagents`
Define at least an implementer and a verifier (different instructions, ideally a
stronger/different model for the verifier). The verifier is the only reason you
can walk away.

### 6. Wire the reach → `loop-connectors`
Add only the connectors the loop actually needs to act (issue tracker, CI, chat,
DB). Each connector is a failure surface — auth expiry, rate limits, half-done
writes. Make every external write idempotent and logged.

### 7. Assemble and dry-run
Compose the pieces. Run **once, attended, with a tiny budget** before scheduling.
Watch what it does. Only then hand it the cadence from step 2.

## Output of this skill

A concrete, runnable loop, not a description of one:
- a state file (from `loop-memory`, e.g. `.loop/PROGRESS.md`),
- a trigger (cron/`/loop`/Action/hook from `loop-automation`),
- agent definitions in `.claude/agents/` (from `loop-subagents`),
- a triage/discovery skill the automation calls,
- explicit budget + kill switch + escape hatch.

A reusable starting point for all of the above lives in this repo's
`templates/` and `examples/nightly-triage-loop.md`.

## The line to keep

The loop changes the work; it does not delete you from it. Two people build the
same loop and get opposite results — one moves faster on work they understand,
the other avoids understanding it. The loop can't tell the difference. Design it
like someone who intends to stay the engineer, not the person who presses go.
