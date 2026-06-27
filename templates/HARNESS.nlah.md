# Harness: <name>
<!--
  A Natural-Language Agent Harness (NLAH): an editable, no-code description of
  run-level harness policy. Fill every section. A reader (human or an IHR-style
  runtime) must be able to point to all five executable elements:
    agent calls · handoffs · state updates · validation gates · artifact contracts
  Keep clauses short, declarative (policy, not procedure), one concern each.
  See the nlah-authoring skill. Based on NLAH, arXiv:2603.25723.
-->

## Purpose & success predicate
- Purpose: <what this harness accomplishes>
- Done when (checkable): <e.g. "verifier returns PASS and the suite stays green">

## Roles & adapters            <!-- agent calls -->
- <role name> — model/host: <e.g. Antigravity> — may: <…> — may NOT: <…> — tools: <…>
- <role name> — model/host: <e.g. Claude>      — may: <…> — may NOT: <…> — tools: <…>
- (Maker and verifier MUST be different roles.)

## Stages & handoffs           <!-- handoffs -->
1. <stage> → produces <artifact> → hands to <role>, passing <state>
2. <stage> → …

## Loops                       <!-- see loop-pattern-spec; exit + budget mandatory -->
- loop <name>: type=<reflection|verifier|learning|meta> · trigger=<…> ·
  body=<…> · exit=<checkable predicate> · budget=<max iters/tokens/time>

## State & memory              <!-- state updates -->
- Persists: <what> · schema: <fields> · location: <file/board> · single source of truth: yes

## Validation gates            <!-- validation gates -->
- gate <name>: check=<command/criteria> · run by=<role, NOT the maker> ·
  pass=<…> · fail=<route to …>

## Artifact contracts          <!-- artifact contracts -->
- <artifact>: required shape=<schema/fields> · acceptance=<criteria>

## Failure recovery
- On gate FAIL: <retry N then escalate to triage/human>
- On error/timeout/stuck loop: <stop + record + escalate>

## Budget & safety
- Ceiling: <max iters / tokens / wallclock per run>
- Kill switch: <how to stop it, e.g. presence of .loop/STOP>
- Blast radius: <branches/PRs only; no destructive or prod ops without a gate>
- Secrets: referenced by name only; values in the tool's secret store

## Observability
- Logged/inspectable: <every external write, each loop's budget line, gate verdicts>
