---
name: harness-generator-studio
description: Scaffold a complete agent harness via a browser studio using only selections (dropdowns/forms) — no code, no install for the build step. Use when the user wants to "generate a harness fast", scaffold a branded CLI harness with built-in loops/memory/MCP, or use the MetaHarness studio (ruvnet/agent-harness-generator) for Claude Code / OpenClaw. Covers the selection workflow, what the generated harness contains, and verifying it before trusting it.
---

# Harness Generator Studio (MetaHarness)

The fastest no-code path: a browser studio that scaffolds a complete, branded
harness from selections in under a minute. No install for the build step; you
make choices in dropdowns/forms and download a ready harness.

> Tool: **MetaHarness** — `ruvnet/agent-harness-generator`. Live client-side
> studio at `https://ruvnet.github.io/agent-harness-generator/` (no backend, no
> telemetry). MIT. Supports **Claude Code** (richest surface) and **OpenClaw**
> among other hosts. Verify current behavior in the studio — it evolves.

## Selection workflow (pure UI)

1. Open the studio URL.
2. **Vertical/domain** — pick a workflow template (coding, research, trading,
   support, legal, etc.) or blank for custom. (Many templates ship.)
3. **Host / agentic CLI** — choose Claude Code or OpenClaw (or another supported
   host). This sets host-specific config.
4. **Branding/name** — name the harness.
5. **Download** the `.zip` containing the full harness.
6. Unzip and run the single launcher command (e.g. `npx <your-harness>`).

To "update," re-run the studio with different selections and regenerate — the
selections are the config surface.

## What the generated harness typically includes

Built in, with no code from you:
- **Memory subsystem** with vector search for session persistence (`loop-memory`).
- A **learning loop** that refines behavior from observed patterns
  (`loop-pattern-spec` — treat as a learning loop: anchor it to a real gate).
- An **MCP server** for cross-host compatibility (`loop-connectors`).
- **Guardrails** and a modular structure.
- A **CLI launcher**, and cryptographic signing/provenance on releases.
- (Studio also exposes extras like template libraries, cost/performance scoring,
  self-optimizing modes, and MCP tool scanning — feature set changes; confirm in
  the studio.)

## Where the generator fits in the no-code track

- It is the **scaffold**, not the design. Run `harness-enquiry` first so you know
  which vertical, host, and loops you actually want — then map those answers to
  studio selections (the enquiry's "generator mapping" output).
- For loop/verification logic the studio doesn't express, capture it in the NLAH
  (`nlah-authoring`) and/or build it on a canvas (`visual-harness-canvas`); use
  the generated harness as the base it plugs into.

## Verify before you trust it (non-negotiable)

A scaffold is a starting point, not a vouched-for system. Before any unattended
run:
- **Read the included guardrails** and set a real budget + kill switch
  (`harness-guardrails`). Don't assume the defaults match your risk tolerance.
- **Confirm an independent verification gate exists.** A learning loop that
  grades itself is not verification — add/point it at an independent check
  (`harness-verification`, `loop-subagents`).
- **Review the MCP tools** it wires in; remove any the harness doesn't need
  (least privilege, `loop-connectors`).
- **Dry-run attended** with a tiny budget and read what it does.

Generated speed is real; ownership is still yours.
