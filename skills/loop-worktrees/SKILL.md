---
name: loop-worktrees
description: Isolation for parallel agents in a loop — use git worktrees so two agents working at once don't clobber each other's files. Use when a loop fans out to multiple agents/subagents, when running parallel tasks on one repo, "run these in parallel without conflicts", or setting isolation:worktree on subagents. Covers when to isolate, creation/cleanup, and the human review ceiling.
---

# Loop Worktrees (isolation)

The moment a loop runs more than one agent, files start colliding — the same
headache as two engineers editing the same lines without talking. A git
worktree fixes it: a separate working directory on its own branch sharing the
same repo history, so one agent's edits can't touch another's checkout.

## When to isolate

- **Isolate** whenever ≥2 agents may write at the same time, or a background run
  writes while you work locally.
- **Don't bother** for strictly serial work, or read-only discovery (triage that
  only reads can run on the local checkout).

## How (Claude Code)

- `--worktree` flag: open a session in its own fresh checkout.
- `isolation: worktree` on a subagent: each helper gets a fresh checkout that
  cleans itself up afterward. This is the main lever inside a loop.
- Raw `git worktree add ../wt-<task> -b loop/<task>` when you need full control.

Pattern for a fan-out loop:

```
for each independent task:
    spawn subagent (isolation: worktree)   # own branch, own files
    → implementer edits in its worktree
    → verifier reviews that worktree's diff
    → on pass: open PR from that branch
    → worktree auto-cleans
```

## Conventions that keep it sane

- One worktree = one task = one branch. Name branches `loop/<short-task-id>` so
  abandoned ones are easy to spot and prune.
- Never let two worktrees target the same branch.
- Prune dead worktrees on a schedule: `git worktree prune` and delete merged
  `loop/*` branches. The `.gitignore` in this repo already ignores `.loop/`
  runtime state so worktrees don't carry machine-local junk.
- Keep the shared state file (`loop-memory`) on the main branch or a dedicated
  branch — not duplicated per worktree, or you'll fork your own memory.

## The real ceiling

Worktrees remove the *mechanical* collision. They do not remove the *human*
one: every parallel branch still needs review, and your review bandwidth — not
the tool — decides how many agents you can actually run. Spinning up ten
worktrees you can't read is not ten units of progress; it's ten units of
unreviewed risk. Scale parallelism to the rate you can actually verify.
