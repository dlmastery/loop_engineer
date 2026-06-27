---
name: loop-pattern-spec
description: Specify a loop's behavior precisely — type, trigger, body, and exit predicate — so a visual tool, generator, or runtime implements the loop you actually meant. Use when describing reflection loops, eval/verifier loops, learning loops, or meta-optimization loops for a harness, or when a "loop" in a spec is vague. Every loop must have an exit condition and a budget, or it is an infinite bill.
---

# Loop Pattern Specification

A "loop" in a spec is useless until its semantics are pinned. This skill makes
each loop unambiguous so a no-code tool (`visual-harness-canvas`,
`harness-generator-studio`) or an NLAH runtime (`nlah-authoring`) implements the
behavior you meant — not a plausible guess.

## Specify every loop with four fields

```
loop <name>:
  type:    reflection | verifier | learning | meta-optimization
  trigger: what starts an iteration
  body:    what happens each iteration (which roles act, what they produce)
  exit:    the checkable predicate that ends the loop   ← never omit
  budget:  max iterations / tokens / time before forced stop  ← never omit
```

`exit` and `budget` are mandatory. A loop with no exit predicate and no budget is
an infinite bill and a runaway — the `nocode-harness` non-negotiable.

## The four loop types

### Reflection loop
The agent critiques and revises its own output before handing off.
- **exit**: self-critique surfaces no new issues, OR max revisions hit.
- **caution**: self-reflection is *not* verification — the maker is still
  grading itself. Always follow a reflection loop with an independent gate.

### Eval / verifier loop (the load-bearing one)
A maker produces; an **independent** verifier passes/fails against a contract;
fail routes back to the maker.
- **exit**: verifier returns PASS against the validation gate
  (`harness-verification`, `loop-subagents`), OR max attempts → escalate to a
  human/triage.
- **rule**: the verifier role is never the maker role. This is the loop that lets
  you walk away.

### Learning loop
The harness refines behavior from observed patterns across runs (the
"learning loop" in generator-scaffolded harnesses).
- **exit**: a target metric plateaus or a run budget is reached.
- **caution**: learning that optimizes a weak metric drifts. Anchor it to the
  same validation gate, and keep a human checkpoint — this is where unattended
  quality silently erodes.

### Meta-optimization loop
A loop that tunes the harness itself (roles, prompts, gate thresholds), e.g. the
self-optimizing modes some generators expose.
- **exit**: harness performance on a held-out eval set stops improving.
- **caution**: highest blast radius — it edits the thing doing the work. Gate
  changes behind a human and a frozen baseline eval; never let it loosen the
  validation gate to make numbers go up.

## Composition
- Nest carefully: a verifier loop *inside* a learning loop is common and fine; a
  meta-loop wrapping everything needs the tightest budget of all.
- Give each loop its **own** budget; the outer budget is not a substitute.
- State what passes between iterations (it's a handoff + state update —
  `nlah-authoring`, `loop-memory`).

## Translating to a visual canvas
- A loop = a back-edge from a downstream node to an upstream one, plus a
  condition node holding the `exit` predicate.
- The `budget` becomes an iteration-count guard / max-loops setting on the loop
  node. If the tool has no native loop-budget control, add a counter state node
  and a condition that exits when it's exceeded. (`visual-harness-canvas`)
