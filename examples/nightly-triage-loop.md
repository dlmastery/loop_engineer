# Worked example: the nightly triage loop

This is the loop from the essay, assembled from the skills in this repo. It runs
once a night, finds work, fixes the safe parts, gets each fix independently
verified, opens PRs, and leaves the rest for a human. You design it once; you do
not prompt any of the steps.

## Shape

```
                      ┌─────────────────────────────────────────────┐
                      │  automation (cron / GitHub Action) 07:00     │  loop-automation
                      └───────────────────────┬─────────────────────┘
                                              ▼
                              calls the `triage` skill           templates/triage-skill
                      reads CI failures · open issues · commits
                                              ▼
                          writes findings → .loop/PROGRESS.md     loop-memory
                          (Next + Triage inbox + budget ledger)
                                              ▼
                 ┌──────────── for each item in `Next` (within budget) ───────────┐
                 ▼                                                                 │
        open isolated worktree  (loop/LP-NN)                          loop-worktrees
                 ▼                                                                 │
        implementer subagent drafts the fix                          loop-subagents │
                 ▼                                                                 │
        verifier subagent: PASS/FAIL vs stop condition + skills      loop-subagents │
                 ▼                                                                 │
        PASS → open PR, link issue, post to Slack                    loop-connectors│
               record evidence in Done                               loop-memory   │
        FAIL → record reason; retry once or escalate to inbox                       │
                 └─────────────────────────────────────────────────────────────────┘
                                              ▼
                   anything unhandled stays in Triage inbox for you
```

## Prerequisites (the harness — do this first)

Run the `harness-prep` skill until every box is green:
- `harness-bootstrap`: clean clone builds + `npm run verify` exits 0.
- `harness-project-skills`: conventions captured in `.claude/skills/`.
- `harness-verification`: `npm run verify` (whole-repo) and scoped checks exist;
  stop conditions written as predicates.
- `harness-guardrails`: budget + `.loop/STOP` kill switch + least-privilege
  tokens + secrets externalized.

A loop on an unprepared harness "fixes" phantom failures. Don't skip this.

## Build steps (the loop — `loop-engineer` skill)

1. **Goal as a predicate.** "For each `Next` item, a verifier confirms the item's
   scoped test passes and `npm run verify` stays green, then open a PR;
   unfixable items go to the triage inbox."
2. **Memory.** `cp templates/PROGRESS.md .loop/PROGRESS.md`, commit it.
3. **Discovery skill.** `cp -r templates/triage-skill .claude/skills/triage`,
   then edit its sources/labels for your repo.
4. **Agents.** `cp templates/agents/*.md .claude/agents/`. Tune models to budget.
5. **Trigger.** Schedule it (out-of-session, since it runs while you sleep):

   ```
   /schedule "every weekday 07:00: run the loop-engineer nightly triage loop on
   this repo — call the triage skill, then for each Next item within budget spawn
   implementer+verifier in a worktree, open PRs on PASS, update .loop/PROGRESS.md,
   stop if .loop/STOP exists."
   ```

   (Or commit a GitHub Actions workflow that runs the same prompt headless.)
6. **Budget + kill switch.** Record the budget in `PROGRESS.md`; confirm the loop
   checks for `.loop/STOP` before each iteration.

## Dry run before you trust it

```bash
# attended, tiny budget, no schedule:
#  - run the triage skill by hand, inspect PROGRESS.md
#  - let it process ONE Next item end to end
#  - read the PR and the verifier's evidence yourself
```

Only after one clean supervised pass do you turn on the 07:00 cadence.

## What you actually did

You designed it once. You did not prompt any of those steps. The state file is
the spine, so tomorrow's run resumes today's. And you stayed the engineer: you
read the PRs, you set the budget, you own "done." The loop moved the leverage
point — it didn't remove you.
