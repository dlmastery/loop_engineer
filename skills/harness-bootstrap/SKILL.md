---
name: harness-bootstrap
description: Make a repo reproducibly buildable and testable by a cold agent — the first step of preparing a harness for loop engineering. Use when "make this repo agent-ready / loop-ready", a fresh checkout doesn't build or test cleanly, or setting up CLAUDE.md and documented build/test commands before automating. A loop on a repo that doesn't build from a clean clone will "fix" phantom failures.
---

# Harness Bootstrap

The harness is the environment one agent runs inside. Before any loop runs on
top of it, a **cold** agent — no conversation history, fresh checkout — must be
able to install, build, run, and test the project from documented commands. If
it can't, every downstream signal the loop reads is noise.

## The cold-clone test

The bar is literal: in a fresh clone, with nothing but what's written down, can
an agent get to green?

```
1. clone
2. <one documented install command>
3. <one documented build command>
4. <one documented test command>  → exits 0, deterministically
```

Run it yourself in a throwaway directory. If any step needs knowledge that lives
only in your head or this conversation, that step is broken for a loop.

## Fix what the cold clone exposes

- **Pin and document deps.** Lockfile committed. Node/Python/etc. version pinned
  (`.nvmrc`, `.tool-versions`, `engines`). The agent should not guess versions.
- **One command per phase.** Wrap multi-step setup in a script or task runner
  (`make setup`, `npm run build`, `npm test`). The loop calls names, not recipes.
- **Deterministic tests.** Flaky tests poison a loop — it can't tell a real
  regression from noise and will thrash. Quarantine or fix flakes *before*
  automating (see `harness-verification` for the pass/fail contract).
- **No interactive prompts** in any agent-run command. Pass `--yes`/non-interactive
  flags; a loop can't answer a prompt.
- **Hermetic-ish env.** Document required env vars (names only) and provide a
  `.env.example`. Real secrets come from the env/secret store (`harness-guardrails`).

## Write CLAUDE.md

`CLAUDE.md` at the repo root is the cold agent's first read. Keep it short and
pointed:

```markdown
# <Project>

## Build & test
- Install: `<cmd>`
- Build: `<cmd>`
- Test (all): `<cmd>`   ← must exit 0 on a clean clone
- Lint/format: `<cmd>`

## Where things are
- <module> → <path>

## Project skills
This repo's conventions and workflows are captured as skills in `.claude/skills/`.
Read the relevant skill before working in an area. (See harness-project-skills.)

## Non-obvious constraints
- <"we don't do X because of Y">
```

CLAUDE.md states the *what/where*; deep conventions and workflows belong in
skills (`harness-project-skills`), not crammed here.

## Done when

- A clean clone reaches green via documented commands, with no hidden steps.
- `CLAUDE.md` exists, is accurate, and points at the project skills.
- No agent-run command blocks on interactive input.

Then move to `harness-project-skills`.
