---
name: loop-subagents
description: The maker/checker split inside a loop — keep the agent that writes the code separate from the agent that checks it. Use when designing subagents/agent teams for a loop, defining .claude/agents, "one explores, one implements, one verifies", adversarial review, or deciding the loop's stop condition. The verifier is the only reason you can walk away. Covers role split, model/effort choice, agent file shape, and token cost.
---

# Loop Subagents (maker ≠ checker)

The most useful structural move in a loop is splitting the one who writes from
the one who checks. The model that wrote the code is far too generous grading
its own homework. A second agent — different instructions, often a different or
stronger model — catches what the first talked itself into.

Because the loop runs while you are not watching, a verifier you actually trust
is the only thing that lets you walk away.

## The standard split

| Role | Job | Bias to counter |
|------|-----|-----------------|
| Explorer (optional) | Read-only recon: where's the relevant code, what patterns exist | — |
| Implementer | Draft the change | "make the diff", optimism |
| Verifier | Check the draft against the spec, tests, and project skills | independence from the maker |

The verifier must be **adversarial and independent**: it did not write the code,
it does not assume the code is right, and it reports against an external standard
(the tests from `harness-verification`, the conventions from project skills),
not against the implementer's intentions.

This is the same primitive as `/goal`: a fresh model decides whether the loop is
done instead of the one that did the work. Maker/checker, applied to the stop
condition itself.

## Choosing models and effort

Spend capability where a second opinion is worth paying for:
- **Verifier / security reviewer**: strong model, high reasoning effort. This is
  where you want the expensive opinion.
- **Implementer**: capable model, normal effort.
- **Explorer**: fast, cheap, read-only.

Each subagent runs its own model and tools, so each one **burns more tokens**.
That's the trade: a real verifier costs tokens; an unreviewed loop costs
trust. Don't add roles you won't use — but never cut the verifier to save
tokens. See `harness-guardrails` for budgeting.

## Defining agents (Claude Code)

Agents live in `.claude/agents/<name>.md` with frontmatter and instructions.
Ready-to-use definitions are in this repo at `templates/agents/`:
`implementer.md` and `verifier.md`. Shape:

```markdown
---
name: verifier
description: Independent reviewer. Checks a diff against tests, lint, and project conventions. Never trusts the implementer's claims.
tools: Read, Grep, Glob, Bash
model: opus
---

You did NOT write this code. Assume nothing works until a command proves it.
1. Run the verification command (see harness-verification).
2. Diff against the spec and project skills; list violations.
3. Output a verdict: PASS (with evidence) or FAIL (with specific, fixable reasons).
Do not fix the code. Do not soften FAIL into "looks mostly fine."
```

Spawn them in parallel when work is independent; fold results back into one
decision. For the human-side rationale see "adversarial code review" and the
"code agent orchestra" patterns.

## Wiring into the loop

```
implementer (in its worktree) → produces a diff
        ↓
verifier (fresh context) → PASS / FAIL against tests + skills
        ↓
PASS → open PR, record evidence in state file (loop-memory)
FAIL → record reason, return to implementer or escalate to triage
```

"Done" is what the verifier certifies, not what the implementer claims. And even
a verifier's PASS is a claim, not a proof — your own review of what shipped is
still the last line. The loop makes "done" mean *more*; it does not make it mean
*everything*.
