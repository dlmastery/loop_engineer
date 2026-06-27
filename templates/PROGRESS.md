# Loop State — <loop name>
<!--
  The spine of the loop. The agent forgets between runs; this file doesn't.
  Read this FIRST every run to decide the next thing; write it LAST to record
  what happened. A run that doesn't update this file didn't happen.
  Copy to .loop/PROGRESS.md and commit it. See the loop-memory skill.
-->
_Last run: <ISO timestamp> · Run #<n>_

## Now (in flight)
<!-- items currently being worked; one worktree/branch each -->
- [ ] LP-12 <description> · worktree loop/LP-12 · <status: implementer done, awaiting verify>

## Next (queued, verified worth doing)
<!-- triaged and confirmed worth doing, not yet started -->
- [ ] LP-13 <description> · from: <source, e.g. nightly triage 2026-06-27>

## Done (verified)
<!-- only items a verifier certified; record the evidence -->
- [x] LP-09 <description> · PR #<n> · verifier: <evidence, e.g. tests green, lint clean>

## Tried & abandoned (do not retry)
<!-- stops the loop relitigating dead ends forever -->
- LP-07 <description> · reverted: <reason + link>

## Triage inbox (needs a human)
<!-- anything the loop can't safely handle; never drop silently -->
- LP-15 <description> · <why the loop punted> · <date>

## Budget ledger
<!-- early warning for a loop burning tokens without closing items -->
- Run #<n>: ~<tokens> tokens · <wallclock> · <items closed> closed · <items escalated> escalated
