---
name: harness-verification
description: Turn "done" into a command that returns pass/fail instead of a claim — the verification step of preparing a harness for loop engineering. Use when defining a loop's stop condition, /goal success criteria, what the verifier subagent checks against, or "make done mean something" before automating. You cannot let a loop decide done until done is a script.
---

# Harness Verification

A loop running unattended is a loop making mistakes unattended. The only thing
that makes its "done" mean anything is an external check the loop cannot
flatter. Your job here: make "done" a **command that returns pass/fail**, not a
sentence an agent writes about its own work.

This is what `/goal` grades against and what the verifier subagent
(`loop-subagents`) checks. It only works if you build it first.

## The verification contract

Define, for the repo as a whole and for each area a loop will touch, a command
that:
- exits **0 on pass, non-zero on fail**,
- is **deterministic** (same input → same verdict; flakes are disqualifying),
- runs **non-interactively**,
- is **fast enough** to run every loop iteration.

```bash
# whole-repo gate (coarse, for broad loops)
npm run verify        # = lint && typecheck && test, exits 0/non-0

# scoped gate (tight, for focused loops — cheaper, sharper)
npm test -- test/auth   # only the area in play
```

Scoped gates matter: a loop fixing auth should verify against `test/auth`, not
the whole suite — faster iterations and a clearer signal.

## Write stop conditions as verifiable predicates

A `/goal` or loop stop condition must be checkable, not vibes:

| Bad (unverifiable) | Good (a command can decide it) |
|--------------------|--------------------------------|
| "the auth flow works" | "`npm test -- test/auth` exits 0 and `npm run lint` is clean" |
| "improve performance" | "benchmark `X` ≥ baseline in `bench/baseline.json`" |
| "fix the bug" | "the failing test `repro_test.ts::case_3` now passes and no other test regresses" |

If you can't express "done" as a command, you're not ready to automate it —
write the failing test or the check first.

## Layers of verification (cheap → expensive)

Run in order; stop at the first failure to save tokens:
1. **Format/lint** — style and obvious mistakes.
2. **Typecheck** — contract violations.
3. **Unit/scoped tests** — behavior in the area touched.
4. **Full suite** — regressions elsewhere.
5. **Verifier subagent** — judgment a script can't encode (design, security,
   "did this actually satisfy the spec"), checking against layers 1–4 plus
   project skills (`loop-subagents`).

Layers 1–4 are mechanical truth. Layer 5 is independent judgment. A loop needs
both: the script catches what's measurable, the verifier catches what's
talked-around.

## The honest caveat

"Done" from this contract is a strong claim, not a proof. Verification is still
ultimately on you — the contract is what lets you trust the loop between your
reviews, not instead of them. Ship code you confirmed works.

## Done when

- One whole-repo command returns a clean pass/fail.
- The areas a loop will touch each have a fast, scoped check.
- Stop conditions for planned loops are written as predicates a command decides.

Then move to `harness-guardrails`.
