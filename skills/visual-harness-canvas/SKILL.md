---
name: visual-harness-canvas
description: Build a harness and its loops on a drag-and-drop node canvas (Langflow or Dify) — no code, configure everything via side panels and dropdowns. Use when the user wants to construct harness logic visually, "build it by dragging boxes", model loops/verification gates/memory as nodes and edges, or realize an NLAH spec in a visual tool. Covers node-to-component mapping, building loops as back-edges, and the dry-run discipline.
---

# Visual Harness Canvas (Langflow / Dify)

Construct the harness by manipulating visual elements — nodes for agents, tools,
memory, conditions, and loops; edges for flow — and configuring each via side
panels, dropdowns, and textareas. Zero scripting. The harness lives in the
visual graph plus node configs.

Use this to *realize* a spec you already designed (`harness-enquiry` →
`nlah-authoring`). Don't free-style the graph; translate the spec node by node so
the canvas stays in sync with the source of truth.

## Tool fit

| Tool | Strongest for | Notes |
|------|---------------|-------|
| **Langflow** | Explicit loop engineering | Node editor; Agent/LLM nodes (pick Claude/Anthropic via dropdown), tools as nodes, loop-backs, condition gates, memory nodes, parallel branches, side playground; export/deploy as API or self-host (Docker) |
| **Dify** | Agentic workflows + RAG | Drag-drop workflow/agent builder, branching, loops, human-in-the-loop gates, strong production observability; cloud or self-host |

Rule of thumb: **Langflow when loop control is the point; Dify when it's a
workflow with RAG and you want built-in observability.**

## Map components → nodes

Drop one node (or sub-flow) per harness component (`harness-component-taxonomy`):

| Component | Node(s) |
|-----------|---------|
| Roles & adapters | Agent / LLM node, model = Claude (dropdown); one per role |
| Stages & handoffs | Edges between role nodes; the edge carries state |
| State & memory | Memory / state node (singular — don't duplicate per branch) |
| Validation gate | A verifier Agent node + a condition node reading its verdict |
| Loop | A back-edge + condition node holding the exit predicate |
| Failure recovery | Branch from the gate's FAIL output → retry or escalate node |
| Observability | The tool's tracing/monitor view; enable it before running |

## Build a verifier loop on the canvas (the core pattern)

```
[Maker Agent: Claude] ──▶ [Artifact] ──▶ [Verifier Agent: Claude, separate node]
                                                   │
                                        [Condition: PASS?]
                                          PASS │      │ FAIL
                                               ▼      ▼
                                          [Output]  [back-edge → Maker Agent]
                                                         │
                                          [Counter / max-loops guard] ──▶ exceeded ──▶ [Escalate]
```

- The verifier is a **separate node** from the maker — that's the maker/checker
  split (`loop-subagents`) drawn as a graph.
- The condition node holds the **exit predicate** from `loop-pattern-spec`.
- Add a **counter/max-loops guard** so the back-edge can't spin forever — the
  visual form of the budget (`harness-guardrails`).

## Configure each node via its panel
- Model + role boundaries in the Agent node's instruction textarea.
- Pass tools/sub-agents by connecting tool nodes, not by code.
- Put secrets in the tool's credential store, never in a node's text field
  (`harness-guardrails`).

## Dry-run discipline
- Use the side playground to run the graph **attended, tiny budget**, before
  exposing it as an API or scheduling it.
- Watch the trace: confirm the loop exits, the gate actually blocks bad output,
  and state persists where you expect.
- Iterate by editing the graph to match the updated spec — keep the NLAH and the
  canvas in agreement; the spec is the source of truth, the graph is a build of it.
