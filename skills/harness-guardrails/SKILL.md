---
name: harness-guardrails
description: Bound a loop's cost, blast radius, and permissions before it runs unattended — the safety step of preparing a harness for loop engineering. Use when setting token/iteration/time budgets, a kill switch, scoping agent write/tool permissions, handling secrets, or "make sure the loop can't run away or do damage". An unattended loop with no budget and broad permissions is the failure mode, not an edge case.
---

# Harness Guardrails

An unattended loop with no budget, broad write permissions, and no audit trail
isn't an edge case — it's the failure mode. Token usage varies wildly and a loop
with no ceiling is a bill with no ceiling. This skill bounds **cost**, **blast
radius**, and **permissions** before the first unattended run.

## 1. Cost ceiling (non-negotiable)

Every loop needs hard limits and a kill switch. No exceptions.

- **Per-run budget**: max iterations, max tokens, and/or max wallclock. When hit,
  the run stops and records where it stopped in the state file (`loop-memory`) —
  it does not silently continue or restart.
- **Kill switch**: one action that halts the loop — disable the schedule, a
  sentinel file the loop checks (`.loop/STOP`), or pausing the routine. Document
  it where you'll find it at 2am.
- **Budget ledger**: every run appends tokens/time/items-closed to the state
  file. A loop burning tokens without closing items is the signal to stop and
  rethink — catch it from the ledger, not the invoice.
- **Right-size the spend**: subagents multiply token cost (`loop-subagents`).
  Spend on the verifier; be frugal on explorers. Use scoped verification
  (`harness-verification`) so iterations are cheap.

```text
# minimal budget contract, recorded with the loop
max_iterations_per_run: 8
max_tokens_per_run:     ~Xk        # tune from the first attended runs
on_budget_exceeded:     stop + record in PROGRESS.md, do not auto-resume
kill_switch:            presence of .loop/STOP halts next iteration
```

## 2. Blast radius

Limit what a single bad iteration can damage.

- **Write to branches, never straight to main.** The loop opens PRs; merging
  stays a human (or a separate, gated) decision.
- **Isolate parallel work** in worktrees so one agent can't corrupt another's
  files (`loop-worktrees`).
- **No destructive ops without a gate**: no force-push, no history rewrite, no
  `rm -rf`, no prod writes from the loop. If a task needs one, it goes to triage.
- **Fail closed**: a connector down or a check ambiguous → stop the item and
  record it, don't improvise (`loop-connectors`).

## 3. Permissions (least privilege)

- Scope agent tool/file permissions to what the loop needs — not blanket allow.
  In Claude Code, configure permissions/hooks in `settings.json` (the
  `update-config` skill helps).
- Connectors get the minimum role: read-only DB roles, repo-scoped tokens, a
  bot identity so actions are attributable and revocable.
- **Secrets live in env / a secret store — never** in skills, agent files,
  `CLAUDE.md`, or the state file. Provide `.env.example` with names only.
  The repo `.gitignore` already excludes `.env*` and `.loop/` runtime state.

## 4. Audit trail

- Log every external write (`loop-connectors`) and every run's budget line. When
  the loop acts while you're away, the log is how you reconstruct what it did.
- Keep findings the loop couldn't handle in a triage inbox, not dropped.

## Done when

- A documented per-run budget and kill switch exist.
- The loop can only write to branches/PRs; destructive and prod ops are gated.
- Permissions and connector roles are least-privilege; secrets are externalized.
- Every run leaves an audit line.

With all four harness skills green, hand off to the `loop-engineer` skill to
build the loop. The guardrails you set here are what let you mean it when you
walk away.
