# 🎯 Behavioral Rules — Steering How the Agent Codes

Response rules shape how the agent *talks*. Behavioral rules shape how it *codes* — what it assumes, how much it builds, what it touches, how it knows it's done.

## The four failure modes
| Failure | Looks like | Counter-rule |
|---|---|---|
| Silent assumptions | picks one reading of an ambiguous request, runs | Think before coding |
| Overengineering | 1000 lines + strategy patterns where 100 would do | Simplicity first |
| Drive-by edits | reformats quotes, adds type hints orthogonal to the task | Surgical changes |
| Vague execution | "I'll review and improve the code" — no done-condition | Goal-driven execution |

## Paste-ready block (drop into CLAUDE.md)
```markdown
## Coding Rules

### Think before coding
- State assumptions explicitly. Uncertain → ask, don't guess.
- Multiple interpretations → present them, don't pick silently.
- Simpler approach exists → say so.
- Confused → stop. Name what's unclear. Ask.

### Simplicity first
- Minimum code that solves the stated problem. Nothing speculative.
- No abstractions for single-use code. No unrequested config/flexibility.
- No error handling for impossible cases.
- 200 lines that could be 50 → rewrite as 50.

### Surgical changes
- Touch only what the task requires. No drive-by refactors/reformatting.
- Match existing style, even when you'd do it differently.
- Remove only orphans YOUR change created. Pre-existing dead code: flag, don't delete.
- Every changed line traces to the request.

### Goal-driven execution
- Convert tasks to verifiable goals before coding:
  - "Fix the bug" → write reproducing test → make it pass
  - "Add validation" → write tests for invalid inputs → make them pass
  - "Refactor X" → tests green before AND after
- Multi-step: plan as `step → verify: check`, one verification per step.
```

## Success criteria beat instructions
LLMs are exceptionally good at looping until they meet a *specific goal*. Give success criteria, not procedures.

| Imperative (weak) | Verifiable (strong) |
|---|---|
| "Make search faster" | "Search p95 < 100ms, via `bin/bench search`" |
| "Fix the auth bug" | "Test: password change invalidates old sessions. Make it pass." |
| "Clean up the user model" | "User model split to ≤500 LOC each. `bin/test` green before and after." |
| "Add rate limiting" | "11th request in 60s returns 429. Test proves it." |

Strong criteria let the agent loop independently — edit, run, check, repeat — without you adjudicating "done."

## 🚀 Proactive — but about the problem, not the diff

"Surgical" cuts scope; **proactive** owns the outcome. They only look contradictory if you conflate *diff size* with *responsibility*. Both belong in `CLAUDE.md`:

```markdown
### Be proactive
- Fix bugs/security issues you see in files you're already editing. Say you did.
- Ship the mechanical guard with the fix — a defect that can recur gets its lint rule/test in the SAME PR.
- If it felt bad, automate it in the same PR: the same three commands again, prints added then
  deleted, setup knowledge in one head, a command that answers with silence instead of an error.
- Finish the whole task. Report what you VERIFIED, not what you assume happened.
- Blocked on part of it? Complete every other part and say exactly what you left out and why.

### Know how to say no
- Requests arrive as solutions. Recover the problem first.
- Nothing breaks by doing nothing → say so and stop. "This shouldn't be built" is a success.
- It already exists here → point at it.
- Premise assumed rather than measured ("it's slow", "it won't scale") → get the number first.
- One sentence. Offer the smaller thing that works. Don't moralize.
```

**Work like someone who has been burned before.** Prefer the boring, proven path. Distrust your own certainty — check the actual value, schema, and behavior, not the remembered one. Diagnose systematically (read the error, reproduce, bisect); never guess-and-check. "I don't know, checking" beats a confident wrong answer. No ego: deleting your own work is a good day.

**Be stubborn about consequences, not preferences.** Calibrate to blast radius and reversibility, never to how many times the ask was repeated — full rules in [../workflow/shipping-doctrine.md](../workflow/shipping-doctrine.md#be-stubborn-about-consequences-not-preferences).

## Self-checks
- **Overengineering:** would a senior engineer call this overcomplicated? Yes → simplify.
- **Surgical:** does every changed line trace to the request? No → revert the strays.
- **Done:** is there a command that proves it works? No → write the test first.

The overengineered version is rarely *obviously* wrong — it follows patterns, handles edge cases, looks professional. The problem is timing: complexity added before it's needed. Simple now, refactor when the requirement actually arrives.

## Scale rigor to the task
| Task size | Rigor |
|---|---|
| Typo, one-liner | Just do it |
| New function, bug fix | Success criteria + surgical diff |
| Feature, refactor, migration | Full: surface assumptions → plan with verification → tests first |

Put these in `CLAUDE.md` (always on, ~30 lines ≈ cheap insurance every session). Compress before pasting — [compressed-config.md](compressed-config.md).
