---
name: harness-enquiry
description: Run a structured AI enquiry that turns a high-level goal into a complete, importable harness spec — no code. Use when the user wants the AI to act as a harness co-designer, "ask me what you need and generate the config/blueprint", or to design a Claude/Antigravity loop harness through conversation. Produces an NLAH doc plus visual-tool or generator instructions. The Q&A replaces manual config writing.
---

# Harness Enquiry (the co-designer)

Use the AI strictly as a harness co-designer: it asks targeted questions, then
emits artifacts ready for a visual canvas, a generator, or an IHR-style runtime.
The enquiry replaces manual config writing — the user answers questions, the AI
produces the spec.

Drive abstraction top-down: **goals → components → constraints.** Don't ask
about nodes before you know the goal; don't finalize the goal without pinning
constraints.

## Phase 1 — Enquiry

Ask in this order. Stop and resolve a blocker before moving on. Don't dump all
questions at once; ask the cluster, integrate the answer, proceed.

1. **Purpose & success.** What should the harness accomplish, and how is "done"
   decided as a checkable predicate? (No predicate → see `harness-verification`;
   you cannot automate an unverifiable goal.)
2. **Loop types.** Which loops? Reflection, eval/verifier, learning, meta-
   optimization? For each, trigger and exit condition. (`loop-pattern-spec`)
3. **Roles & adapters.** Which models/agents play which role? (e.g. Antigravity
   proposes/executes, Claude verifies.) What are each role's boundaries?
4. **Verification gates.** What independent check passes/fails each stage? Who
   (which role) runs it — must not be the maker.
5. **Memory & state.** What persists between runs and where? What's the state
   schema? (`loop-memory`)
6. **Coordination.** Single agent, sequential handoffs, or parallel swarm? How
   do handoffs pass state?
7. **Safety & budget.** Token/iteration/time ceiling, kill switch, blast-radius
   limits, secret handling. (`harness-guardrails`)
8. **Observability.** What must be logged/inspectable when it runs unattended?

Use `harness-component-taxonomy` as the completeness checklist — every part it
lists must have an answer or an explicit "n/a, because…".

## Phase 2 — Generation

Produce all three, so the user can take any path:
- **NLAH spec** — the editable natural-language harness doc (`nlah-authoring`,
  template at `templates/HARNESS.nlah.md`). The portable source of truth.
- **Visual blueprint** — step-by-step node/edge instructions for Langflow or
  Dify ("add Agent node → set Claude model → connect to verification sub-flow →
  loop-back edge until gate passes → attach memory node"). (`visual-harness-canvas`)
- **Generator mapping** — the MetaHarness studio selections that get closest,
  if scaffolding from a template. (`harness-generator-studio`)

## Phase 3 — Load & dry-run
Hand the artifacts to the chosen tool. Run once attended, tiny budget. Read what
it does before trusting it.

## Phase 4 — Update
Take feedback as spec edits, not code patches: "tighten gate on X", "add a
parallel learning loop", "update the adapter for new Antigravity capability."
Revise the NLAH/blueprint, re-import, and version the output. Keep the harness
logic externalized and optimizable.

## Anti-patterns
- Generating a spec before "done" is a checkable predicate.
- Letting the maker role also be the verifier (the enquiry must force a split).
- Producing a graph with a loop that has no exit condition or budget.
- Asking implementation (which node) before goal and constraints are settled.
