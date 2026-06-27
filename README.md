# loop_engineer

> Stop prompting the agent. Design the system that prompts it.

Loop engineering is replacing yourself as the person who types the next prompt.
You build a small system that finds the work, hands it out, checks it, writes
down what's done, and decides the next thing — then you let that system poke the
agents instead of you. A **loop** is a recursive goal: define a purpose and the
agent iterates until a verifiable stop condition holds.

This repo is an installable **Claude Code plugin** of skills for doing exactly
that — and for preparing the harness the loop runs inside. The skills also work
standalone (copy into `~/.claude/skills/` or a project's `.claude/skills/`).

## What's in here

Two skill sets, each with a composite meta-skill that orchestrates the rest,
plus runnable templates.

### The loop (`loop-engineer` meta-skill)
The five building blocks of a loop, plus the memory that is its spine:

| Skill | Block | Job |
|-------|-------|-----|
| **`loop-engineer`** | — | Composite: design a whole loop; refuses one with no verifier or no budget |
| `loop-automation` | Heartbeat | Schedule discovery/triage (`/loop`, `/goal`, cron, hooks, Actions) |
| `loop-worktrees` | Isolation | Git worktrees so parallel agents don't collide |
| `loop-memory` | Spine | Durable on-disk state: done / in-flight / next |
| `loop-subagents` | Maker/checker | One agent drafts, an independent one verifies |
| `loop-connectors` | Reach | Act in real tools (issues, CI, Slack) via MCP |

### The harness (`harness-prep` meta-skill)
The floor the loop stands on — the environment a single agent runs inside:

| Skill | Guarantees |
|-------|-----------|
| **`harness-prep`** | Composite: drive the repo to loop-ready, in order |
| `harness-bootstrap` | A cold agent can build + test from a clean clone |
| `harness-project-skills` | Project intent is written down, not re-guessed each run |
| `harness-verification` | "Done" is a command that returns pass/fail |
| `harness-guardrails` | Cost, blast radius, and permissions are bounded |

### The no-code track (`nocode-harness` meta-skill)
Build, configure, and iterate a harness **without writing or editing code** —
via structured enquiry, an editable natural-language spec, visual canvases, and
browser generators. Same non-negotiables as the code track (independent verifier
+ cost ceiling); reached by declarative and visual means.

| Skill | No-code job |
|-------|-------------|
| **`nocode-harness`** | Composite: run the whole no-code path, enquiry → spec → realize → iterate |
| `harness-enquiry` | AI co-designer Q&A that turns goals into an importable spec |
| `harness-component-taxonomy` | The part list that makes a spec complete |
| `nlah-authoring` | Write the harness as a Natural-Language Agent Harness doc |
| `loop-pattern-spec` | Pin each loop: type, trigger, body, exit, budget |
| `visual-harness-canvas` | Realize it as nodes + edges in Langflow / Dify |
| `harness-generator-studio` | Scaffold a ready harness in the MetaHarness browser studio |

Grounded in real tools/research: NLAH (arXiv:2603.25723), MetaHarness
(`ruvnet/agent-harness-generator`), Langflow, Dify, and the harness-engineering
literature (`ai-boost/awesome-harness-engineering`). The IHR runtime from the
NLAH paper is research — the NLAH doc is treated as a portable source of truth
you feed into a visual tool or generator, not an assumed shipping runtime.

### Templates (`templates/`) — ready to copy
- `PROGRESS.md` — the state-file (memory) schema.
- `triage-skill/SKILL.md` — discovery/triage pass an automation calls.
- `agents/implementer.md`, `agents/verifier.md` — the maker/checker pair.
- `HARNESS.nlah.md` — a fillable Natural-Language Agent Harness spec (no-code).

### Example (`examples/`)
- `nightly-triage-loop.md` — the whole thing assembled end-to-end.

## Quick start

```bash
# 1. Prepare the harness FIRST (a loop on an unprepared repo "fixes" phantoms)
#    → invoke the harness-prep skill; drive every readiness box to green.

# 2. Build the loop
#    → invoke the loop-engineer skill; it walks the 7-step design procedure.

# 3. Scaffold from templates
cp templates/PROGRESS.md            .loop/PROGRESS.md
cp -r templates/triage-skill        .claude/skills/triage
cp templates/agents/*.md            .claude/agents/

# 4. Dry-run attended with a tiny budget, read the output, THEN schedule it.
```

### Install as a plugin
Point your Claude Code plugin config at this repo (it ships a
`.claude-plugin/plugin.json`), or vendor the `skills/` you want into
`.claude/skills/`.

## The two rules this repo will not let you skip

1. **The verifier is not the maker.** If the only judge of "done" is the agent
   that wrote the code, "done" means nothing. The `loop-engineer` skill refuses
   to finish a loop without an independent check.
2. **Every loop has a cost ceiling and a kill switch.** Token usage varies
   wildly; an unbounded loop is an unbounded bill. Set a budget before the first
   unattended run.

## The line to keep

Two people build the same loop and get opposite results. One uses it to move
faster on work they understand deeply; the other uses it to avoid understanding
the work at all. The loop can't tell the difference — you can.

**Build the loop. Stay the engineer.**

## License

MIT — see [LICENSE](LICENSE).
