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
