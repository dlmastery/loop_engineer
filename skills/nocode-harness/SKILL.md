---
name: nocode-harness
description: Composite meta-skill for building and iterating an agent harness with NO code — via structured AI enquiry, a natural-language harness spec (NLAH), visual node canvases (Langflow/Dify), and browser generators (MetaHarness). Use when the user wants to design/configure/update a harness or its loops "without writing code", "no-code", "visually", "by filling boxes", or "generate a config/blueprint I can import". Orchestrates harness-enquiry, harness-component-taxonomy, nlah-authoring, loop-pattern-spec, visual-harness-canvas, and harness-generator-studio. Keeps the same non-negotiables as code harnesses: an independent verifier and a cost ceiling.
---

# No-Code Harness

Build the harness — the execution system around the model that organizes a task
run — and its loops without writing or editing code. The harness logic lives in
UI selections, a visual graph, or a declarative natural-language spec, not in a
script.

This is the no-code track of this repo. The code/process track (`loop-engineer`,
`harness-prep`) still defines *what good looks like*; this track reaches the same
place by declarative and visual means. The principles do not relax.

## The two non-negotiables still apply

No-code does not mean no-discipline. Refuse to call a harness done without:
1. **An independent verifier** — a validation gate the *maker* doesn't grade
   (see `loop-subagents`, `harness-verification`). In a no-code harness this is a
   verification node / a validation-gate clause in the NLAH.
2. **A cost ceiling + kill switch** — max iterations/tokens/wallclock and a stop
   control (`harness-guardrails`). Visual loops with no exit condition are
   infinite bills.

## Component skills

| Skill | No-code job |
|-------|-------------|
| `harness-enquiry` | Run the structured Q&A that turns goals into a complete, importable spec |
| `harness-component-taxonomy` | The checklist of parts every harness needs — so the spec is complete |
| `nlah-authoring` | Write the harness as an editable Natural-Language Agent Harness doc |
| `loop-pattern-spec` | Specify the loop types precisely (reflection / verifier / learning / meta) |
| `visual-harness-canvas` | Realize the spec as nodes + edges in Langflow or Dify |
| `harness-generator-studio` | Scaffold a ready harness via the MetaHarness browser studio |

## The no-code workflow

```
goals ──▶ harness-enquiry ──▶ NLAH spec (nlah-authoring + taxonomy + loop-pattern-spec)
                                   │
                 ┌─────────────────┼───────────────────────────┐
                 ▼                 ▼                             ▼
   visual-harness-canvas    harness-generator-studio     hand to an IHR-style
   (Langflow / Dify graph)  (MetaHarness .zip)           runtime (research; verify)
                 │                 │
                 ▼                 ▼
        attended dry-run with a tiny budget ──▶ iterate by editing the spec/graph
```

### 1. Enquire, don't guess → `harness-enquiry`
Treat the AI as a harness co-designer. It asks targeted questions (purpose,
loop types, verification gates, memory strategy, stop conditions, coordination,
safety, budget) and produces the first complete spec. The questions replace
manual config writing.

### 2. Make the spec complete → `harness-component-taxonomy`
Check the answers against the part list (roles, loop/stage structure, state &
memory, adapters, validation gates, failure recovery, observability). A spec
missing a part is a harness with a hole an agent will fill by guessing.

### 3. Write it as a portable spec → `nlah-authoring`
Capture it as an NLAH: an editable document of run-level policy that an
IHR-style runtime interprets into agent calls, handoffs, state updates,
validation gates, and artifact contracts. A fillable template is at
`templates/HARNESS.nlah.md`. This is the most no-code artifact — plain editable
text, versionable, diffable.

### 4. Pin the loop semantics → `loop-pattern-spec`
For each loop, state its type, trigger, body, and exit predicate explicitly so a
visual tool or runtime implements the behavior you meant.

### 5. Realize it → `visual-harness-canvas` or `harness-generator-studio`
Build the graph by dragging nodes and filling side panels (Langflow/Dify), or
scaffold a branded harness by selections in the MetaHarness studio.

### 6. Dry-run, then iterate declaratively
Run once attended with a tiny budget. Then update by **editing the spec/graph**,
not by patching code: "tighten the verification gate on metric X", "add a
learning loop", "swap the adapter for Antigravity". Re-import. Version every
revision.

## The line to keep
No-code lowers the cost of *building* the harness; it does not lower the cost of
*owning* it. You still set the budget, you still read what shipped, you still own
"done." Design the spec like someone who intends to stay the engineer.
