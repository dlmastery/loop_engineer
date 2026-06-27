---
name: harness-component-taxonomy
description: The completeness checklist of parts every agent harness needs — use to make a harness spec whole before building it, or to choose the right nodes in a visual tool. Use when designing/reviewing a harness, steering an enquiry, or asking "what am I missing?". Covers roles & boundaries, loop/stage structure, state & memory, adapters, validation gates, artifact contracts, failure recovery, and observability. A spec missing a part is a hole an agent fills by guessing.
---

# Harness Component Taxonomy

A harness is more than a model and a prompt. This is the part list — the
checklist that makes a spec *complete* before you build it, and the map for
which nodes to drop on a visual canvas. Drive the `harness-enquiry` against it;
review any NLAH (`nlah-authoring`) against it. Every component below must have an
answer or an explicit "n/a, because…".

## The components

| # | Component | The question it answers | Maps to |
|---|-----------|-------------------------|---------|
| 1 | **Roles & boundaries** | Who acts, and what may each role do / not do? | NLAH agent calls; `loop-subagents` |
| 2 | **Loop / stage structure** | What is the ordered flow, and which parts repeat? | NLAH stages + loops; `loop-pattern-spec` |
| 3 | **State & memory** | What persists between steps/runs, in what schema, where? | NLAH state updates; `loop-memory` |
| 4 | **Adapters** | How does each host/model plug in (Claude, Antigravity, OpenClaw)? | NLAH roles; `loop-connectors` |
| 5 | **Validation gates / contracts** | What independent check passes/fails each stage? | NLAH validation gates; `harness-verification` |
| 6 | **Artifact contracts** | What shape must each stage's output take? | NLAH artifact contracts |
| 7 | **Failure recovery** | What happens on gate-fail, error, timeout, or stuck loop? | NLAH failure recovery |
| 8 | **Observability** | What is logged/inspectable while it runs unattended? | NLAH observability; `harness-guardrails` |
| 9 | **Budget & safety** | Ceiling, kill switch, blast radius, secrets, permissions? | `harness-guardrails` |

## How to use it

- **Steering an enquiry:** walk the table top to bottom. Each row is a question
  cluster. A row with no answer is a gap; surface it before generating the spec.
- **Choosing visual nodes:** each component is a node type to look for —
  agent/LLM nodes (1, 4), loop/condition nodes (2), memory/state nodes (3),
  verification/branch nodes (5, 6), error-handling nodes (7), logging/monitor
  nodes (8). (`visual-harness-canvas`)
- **Reviewing a spec:** if a reader can't locate each component, the spec is
  incomplete — agents fill missing components with confident guesses, which is
  exactly the failure no-code is supposed to prevent.

## Separation rules that keep a harness analyzable

- **Maker ≠ checker** (rows 1 and 5): the role that produces an artifact never
  runs its own validation gate.
- **State is singular** (row 3): one source of truth, not per-branch copies.
- **One concern per component**: so any single part can be edited or ablated
  without disturbing the rest — the modularity that makes harnesses inspectable.

## Minimum viable harness
The smallest spec that is still safe to run unattended must have, at least:
roles with boundaries (1), an explicit loop with an exit condition (2), state
(3), an independent validation gate (5), and a budget + kill switch (9). Anything
less is not loop-ready.
