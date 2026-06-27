---
name: triage
description: Discovery + triage pass for a loop — reads recent CI failures, open issues, and recent commits, then writes findings into the loop state file and the triage inbox. Use when an automation needs to surface and prioritize work at the start of a loop run. Adapt the sources and filters to your repo before scheduling.
---

# Triage (loop discovery pass)

This is the skill an automation calls at the top of a loop run (see the
`loop-automation` skill). It surfaces work and records it; it does **not** fix
anything. Adapt the sources, filters, and connectors to your repo before
scheduling.

## Procedure

1. **Read the spine first.** Open `.loop/PROGRESS.md`. Note what's already in
   `Now`, `Next`, `Done`, and `Tried & abandoned` so you don't re-surface
   resolved or dead-end work.

2. **Gather signals** (read-only; use connectors from `loop-connectors`):
   - CI: failures since the last run.
   - Issue tracker: new/updated issues matching the loop's scope (e.g. a label).
   - Git: commits since the last run that touched watched paths.

3. **Filter to in-scope, actionable findings.** Drop anything:
   - already tracked in the state file,
   - in `Tried & abandoned`,
   - outside this loop's declared scope,
   - that needs a human decision → route to the triage inbox instead.

4. **Prioritize.** Rank by impact × confidence. A broken build > a lint nit.

5. **Write findings back** to `.loop/PROGRESS.md`:
   - actionable + clear → `Next`, with a stable id and `from:` source,
   - actionable but risky/ambiguous → `Triage inbox`,
   - update the budget ledger line for this run.

6. **If nothing actionable was found**, say so and exit quietly. A "nothing
   found" run should be cheap and silent — don't manufacture work.

## Guardrails

- Read-only. This skill never edits code, opens PRs, or merges. The implementer
  and verifier subagents do that downstream (`loop-subagents`).
- Respect the run budget (`harness-guardrails`); if signal-gathering blows the
  budget, record partial results and stop.
- Stable ids: reuse an existing item's id if a finding matches it; never create a
  duplicate (`loop-connectors` idempotency).
