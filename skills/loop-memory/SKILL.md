---
name: loop-memory
description: The spine of a loop — durable on-disk state that survives between runs so the loop resumes instead of restarting. Use when setting up a loop's state/progress file, a triage board, or "remember what's done and what's next" across runs. The model forgets everything between runs; the repo doesn't. Covers what to store, the state-file schema, and update discipline.
---

# Loop Memory (the spine)

A long-running agent forgets everything between runs. So the memory has to live
on disk, not in the context. This sounds too dumb to matter and it is the single
thing every durable loop depends on: the agent forgets, the repo doesn't.

The memory is the spine — it holds what's done, what's in flight, and what's
next, so tomorrow's run picks up where today's stopped instead of re-deriving
the world.

## Where it lives

Pick one and commit to it:
- **A markdown file in the repo** (default: `.loop/PROGRESS.md`). Versioned,
  diffable, reviewable in PRs. Best for most loops.
- **An external board** (Linear, GitHub Issues/Projects). Best when humans and
  the loop share the same queue. Reached via `loop-connectors`.

Keep it singular. One source of truth. Don't fork it per worktree (see
`loop-worktrees`) — the loop must read and write the *same* memory each run.

## What to store (and what not to)

Store the *state of the work*, not a transcript:
- each work item: id, one-line description, status, where it came from,
- what was **tried** and the **outcome** (so the loop doesn't retry a dead end),
- what passed verification and the evidence (test name, PR link),
- what's still open and what's blocked + why,
- a triage inbox for items needing a human.

Do **not** store: secrets, full logs, or anything reconstructable from git. The
state file is a map, not the territory.

## Schema

A ready-to-copy template is at `templates/PROGRESS.md`. Shape:

```markdown
# Loop State — <loop name>
_Last run: <ISO timestamp> · Run #<n>_

## Now (in flight)
- [ ] LP-12 Fix flaky auth test · worktree loop/LP-12 · implementer done, awaiting verify

## Next (queued, verified worth doing)
- [ ] LP-13 Upgrade lint config · from: nightly triage 2026-06-27

## Done (verified)
- [x] LP-09 Null-check in parser · PR #341 · verifier: tests green, lint clean

## Tried & abandoned (do not retry)
- LP-07 Cache layer · reverted: broke SSR, see PR #338 discussion

## Triage inbox (needs a human)
- LP-15 Intermittent prod 500 · loop can't repro · 2026-06-27

## Budget ledger
- Run #<n>: ~<tokens> tokens, <wallclock>, <items closed>
```

## Update discipline (the part loops get wrong)

- **Read it first, write it last.** Every run opens by reading the state file to
  decide the next thing, and closes by recording what happened. A run that
  doesn't update the memory is a run that didn't happen, as far as tomorrow knows.
- **Append outcomes, don't overwrite history.** "Tried & abandoned" is what stops
  the loop relitigating the same dead end forever.
- **One writer at a time.** If parallel agents update state, funnel writes
  through the orchestrator or use an external board with atomic updates — concurrent
  markdown edits corrupt the spine.
- **Log the budget ledger every run.** It's your early warning for a loop that's
  burning tokens without closing items (`harness-guardrails`).
