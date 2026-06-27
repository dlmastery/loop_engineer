---
name: nlah-authoring
description: Write a harness as a Natural-Language Agent Harness (NLAH) — an editable plain-text document that describes run-level harness policy instead of hard-coded controller logic. Use when capturing harness/loop logic declaratively, "write the config as natural language", making control logic portable/inspectable, or producing a spec to feed a visual tool or runtime. Based on the NLAH paper (arXiv:2603.25723). Covers the document sections and the five things a runtime interprets.
---

# NLAH Authoring (declarative, no-code)

A harness is the execution system around a model that organizes a task run.
Usually that logic is buried in tightly-coupled controller code — hard to
inspect, compare, transfer, or ablate. A **Natural-Language Agent Harness
(NLAH)** is the same logic written as an editable document of *run-level policy*.
This is the most no-code artifact in the pack: plain text you can read, diff, and
hand to a teammate or a runtime.

> Grounding: Natural-Language Agent Harnesses, Pan, Zou, Guo, Ni, Zheng —
> arXiv:2603.25723 (Mar 2026). NLAHs are editable docs describing run-level
> harness policy; the **Intelligent Harness Runtime (IHR)** interprets them into
> agent calls, handoffs, state updates, validation gates, and artifact contracts.
> The IHR is research — treat a shipping runtime as unverified. Even without one,
> the NLAH is the source of truth you translate into a visual graph
> (`visual-harness-canvas`) or generator selections (`harness-generator-studio`).

## What a runtime must be able to read out of your NLAH

Write so each of these five is explicit and unambiguous — they are what gets
executed:

1. **Agent calls** — which role/model is invoked, with what instruction and tools.
2. **Handoffs** — when control passes between roles, and what travels with it.
3. **State updates** — what is written to memory/state at each step (`loop-memory`).
4. **Validation gates** — the pass/fail checks between stages, and who runs them
   (never the maker — `harness-verification`, `loop-subagents`).
5. **Artifact contracts** — the required shape of each stage's output (the
   "definition of done" for an artifact: schema, fields, acceptance criteria).

If a reader can't point to each of the five in your document, it isn't an
executable harness policy yet — it's prose.

## Document structure

A fillable template is at `templates/HARNESS.nlah.md`. Sections:

```markdown
# Harness: <name>
## Purpose & success predicate      # checkable "done"
## Roles & adapters                 # each role: model, boundaries, tools  (agent calls)
## Stages & handoffs                # ordered stages; what passes between   (handoffs)
## Loops                            # per loop: type, trigger, body, exit   (loop-pattern-spec)
## State & memory                   # what persists, schema, where          (state updates)
## Validation gates                 # per gate: check, runner, pass/fail    (validation gates)
## Artifact contracts              # per artifact: required shape + accept  (artifact contracts)
## Failure recovery                 # on gate-fail / error / timeout
## Budget & safety                  # ceiling, kill switch, blast radius
## Observability                    # what is logged/inspectable
```

## Authoring rules

- **Declarative, not imperative.** Describe *policy* ("the verifier rejects any
  diff that lowers coverage"), not procedure in disguise. Shorter is the goal —
  the NLAH win is exposing a much smaller, analyzable policy than code.
- **One concern per clause.** So a single module can be ablated/edited without
  touching the rest (the paper's modularity claim).
- **Name every role's boundary.** What it may and may not do. Unbounded roles get
  filled with confident guesses.
- **Make gates independent.** State explicitly which role runs each gate and that
  it is not the producer.
- **No secrets in the document.** Reference them by name; real values live in the
  tool's secret store (`harness-guardrails`).

## Iterating
Update the harness by editing the NLAH and re-importing — not by patching a
generated graph by hand (edits there drift from the source of truth). Version
each revision so behavior changes are reviewable like any other diff.
