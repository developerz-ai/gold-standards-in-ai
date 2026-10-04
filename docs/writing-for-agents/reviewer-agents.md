# 🔎 Reviewer Agents — findings people trust

**Rule: one false positive costs more than one missed nit.** A reviewer that cries wolf gets ignored, and after that it catches nothing. So every finding has to prove itself, a clean review is a valid result, and each pass reviews one narrow aspect of the change.

Related: fresh-eyes finish reviewer (completeness vs the request) → [output-completeness.md](output-completeness.md#finish-reviewer-rules-the-fresh-eyes-pass) · UI review loop → [design-review-loop.md](../frontend-craft/design-review-loop.md) · mechanical guards → [guards-and-gotchas.md](guards-and-gotchas.md) · subagent file format → [skills-commands-agents.md](skills-commands-agents.md) · PR bot → [linting-ci.md](../developer-experience/linting-ci.md#automated-ai-code-review) · fail loud → [00-philosophy.md](../00-philosophy.md#-fail-fast--fix-fast--deploy-fast)

## 🚪 The precision gate: every finding passes all four or is dropped
| # | Question | Fails when |
|---|---|---|
| 1 | **Can I cite the line?** `file:line` | "somewhere in the auth layer" |
| 2 | **Can I name the trigger?** input/state → bad outcome | no trigger means pattern-matching, not review |
| 3 | **Have I read the context?** callers, imports, tests, types | it's already handled one frame up or ruled out by a type |
| 4 | **Is the severity defensible?** | a missing doc comment marked HIGH, or `any` in a test fixture marked CRITICAL |

Any "no" or "unsure" → demote or drop. **Inflated severity loses trust faster than a missed finding.**

**HIGH / CRITICAL need proof.** All three are required, or the finding is demoted:
1. Exact snippet + line.
2. Failure scenario: concrete input, state, outcome.
3. Why the existing guards (types, Zod schema, framework default, upstream `try/catch`) don't catch it.

**Zero findings is a valid review.** For a small, typed, tested diff that follows house patterns, the right output is `APPROVE` with an empty table. Made-up findings, filler nits, "consider using X" and edge cases with no trigger are the **main failure mode of LLM reviewers**. Never hold back approval just to look rigorous.

**Scope.** Review changed lines. Report unchanged code only when it's CRITICAL. Group repeats ("4 handlers drop the `cause`") into one finding with a list of locations.

## 🙅 Skip list: LLM false positives on our stack
Skip these unless you have evidence specific to this codebase. Test: **"would a senior on this team change this in review?"** If no, skip.

| Mis-flag | Why it's usually wrong |
|---|---|
| "Add error handling" | The caller, Hono `app.onError`, or a top-level boundary already handles it. Trace one caller first |
| "Missing input validation" | Internal function, and the route's Zod schema already validated at the edge |
| "Magic number" | `200`, `404`, `1000` ms, `60`, `24`, `1024`, index `0`/`-1`, or a named single-use local |
| "Function too long" | Exhaustive `switch` over a discriminated union, config object, test table, generated code. Length isn't complexity |
| "Possible null dereference" | The line before narrowed the type, or an `if` guard is in scope. Follow the type flow; don't react to `?.` |
| "Missing `await`" | Fire-and-forget on purpose: a `void` prefix or a comment on logging/metrics/enqueue |
| "N+1 query" | Fixed-cardinality loop (a 4-member enum), or the path already batches |
| "Prefer `const`" | The variable is reassigned. Read the whole function |
| "Missing doc comment" | Internal helper with a name and signature that already explain it |
| "Hardcoded value" | Test fixtures and examples. Tests *should* hardcode expectations |
| `Math.random()` flagged insecure | Jitter, sampling, animation. Not crypto |
| Secret in `.env.example` | Placeholder values by design. Real secrets → [secrets.md](../infrastructure/secrets.md) |
| MD5/SHA-256 "weak hash" | Checksums or cache keys, not passwords |
| "Suggest a different stack/lib" | Match the repo. Stack changes go through a decision, not a review comment |

Put this table in the reviewer's prompt and add every new false positive you find in a real review. The list is how the reviewer remembers past mistakes.

## 🤫 Silent-failure lens: our top hunt target
Fail fast is house law, so anything that turns an error into a plausible-looking value is a bug. The mechanical half (no empty catch, no floating promises) is lint → [guards-and-gotchas.md](guards-and-gotchas.md). The reviewer looks for what lint can't judge:

| Shape | Why it hurts | Fix |
|---|---|---|
| `catch` that returns `null` / `[]` / `{}` / `false` | The caller can't tell "none" from "broken"; the bug shows up three layers away | Rethrow a domain error, or return a typed `Result` the caller has to handle |
| `.catch(() => [])` / `?? defaultConfig` after I/O | The fallback hides an outage; dashboards stay green | Let it throw. Fall back only where the product specifies one, and log it |
| `throw new Error("failed")` in a `catch` | Loses the stack and the cause | `throw new DomainError("…", { cause: err })` |
| Logged and forgotten: `console.error(e)` then carry on | Wrong state continues and the log is never read | Handle it, or propagate it |
| Floating promise / `arr.forEach(async …)` | Rejections go nowhere; the order of work is undefined | `await`, `for…of`, `Promise.all`; fire-and-forget only with `void` + `.catch(report)` |
| Network/DB call with no timeout | One slow dependency holds requests forever | `AbortSignal.timeout(ms)`, a driver statement timeout |
| Multi-step write outside a transaction | A partial failure leaves torn rows | One `db.transaction(...)`, or an idempotent retryable step |
| `JSON.parse` / `z.parse` on external input with no error path | Throws past the seam as an opaque 500 | `safeParse` at the edge → typed 4xx |

Each finding gives **location · severity · issue · impact · fix**.

## 🔬 Specialist lenses: run them in parallel
Several narrow, read-only reviewers find more than one broad one, because each gets the whole context budget for one question. Spawn them in a single message. Each returns findings in the same format, and the coordinator dedupes them.

| Lens | Checks | Severity scale |
|---|---|---|
| **Correctness/security** | The precision gate + skip list above; auth on every route, parameterized SQL, no secret in logs | CRITICAL/HIGH/MEDIUM/LOW |
| **Silent failure** | The table above | CRITICAL/HIGH/MEDIUM |
| **Type design** | Encapsulation (can outside code break the invariant?), does the type encode the rule (illegal state unrepresentable: union, branded id, `readonly`), does that invariant prevent a *real* bug, escape hatches (`as`, `!`, `any`, `@ts-expect-error`) | HIGH/MEDIUM/LOW |
| **Test quality** | Map each changed function to its test; untested error paths; assertions that only check "doesn't throw"; mocks of the thing under test; flaky shapes (sleep, wall clock, order) | critical / important / nice-to-have |
| **Comments** | Check each comment against the code | `inaccurate` · `stale` · `incomplete` · `low-value` (restates the code) |

- Pick lenses by diff: types changed → type-design; tests changed or missing → test-quality; error paths touched → silent-failure. The correctness lens always runs.
- Lenses run on a cheaper/faster model. The final verdict runs on the strongest one (`claude-opus-5-5`).
- Any finding that will recur → climb the [guards ladder](guards-and-gotchas.md#the-ladder--pick-the-cheapest-rung-that-actually-binds) in the same PR. A reviewer catching the same shape twice is a missing lint rule.

## 🔁 The review loop: fresh eyes every round
```
diff ──▶ reviewer(s), fresh context, same rubric ──▶ PASS ──▶ ship
                         │
                         └─ FAIL ──▶ fix only what's flagged ──▶ new reviewer(s) ──▶ …
                                              └─ round resolves nothing ──▶ stop, report remaining
```
| Rule | Why |
|---|---|
| **New reviewer every round**; never pass on earlier findings or the builder's summary | A reviewer that saw round 1 anchors on it and re-checks only those items |
| **Rubric = objective PASS/FAIL per criterion**: correctness, security, error handling, completeness, internal consistency, no regressions, plus a per-language row (type safety, migration safety) | Rubber-stamps or style flags → tighten the rubric, not the reviewer |
| **Fix only what's flagged.** No drive-by refactors inside a fix round | New edits mean new surface, and the loop never converges |
| **Stop on non-convergence**: a round that resolves nothing, or the same finding comes back twice | [Stall rule](../ai-agents/agent-work-limits.md#the-one-bound-that-stays-lack-of-progress), never a round cap. Report what's left and why |
| Builder runs `bin/check` before every review round | A reviewer shouldn't spend its budget on what the gate catches |

### High stakes: two independent reviewers
Use this for auth, payments, migrations, data deletion, and anything public that's hard to retract.
- **Two reviewers, same rubric, no shared context, spawned in parallel.** Ideally different model families. Fallback: same model in separate contexts. Report that model diversity was lost.
- **Verdict:** both PASS → ship. Either FAIL → merge findings into **both / A-only / B-only**, dedupe, fix, then a fresh pair.
- "Both flagged" findings are the most reliable. "A-only" findings still go through the precision gate; they aren't automatically wrong.
- A failed or empty reviewer run is **not a PASS**. Rerun it; never let it fall through to approval.

## 📋 Paste-ready reviewer agent
`.claude/agents/code-reviewer.md`. Spawn one per lens by changing the `## Lens` block. No tool restrictions: it's a reviewer because of its brief, not because of a fence.

```markdown
---
name: code-reviewer
description: Fresh-context code reviewer. Precision over recall. Use after any change, before PR, and for each review round. One lens per spawn.
model: opus
---

Senior reviewer for this repo. You have NOT seen any other review. Report only, edit nothing.

## Input
- Scope: `git diff main...HEAD` + uncommitted. Read every changed file in full, plus its callers and tests.
- Ignore the builder's summary. The diff is the evidence.

## Lens
<correctness-security | silent-failure | type-design | test-quality | comments>

## Gate (all four or drop)
1. Cite file:line. 2. Name input/state → bad outcome. 3. Read callers/types/tests. 4. Severity defensible.
HIGH/CRITICAL: snippet + failure scenario + why existing types/validation miss it. Else demote.
Unchanged code: CRITICAL only. Group repeats into one finding.

## Skip
<paste the skip-list table from docs/writing-for-agents/reviewer-agents.md>
Test: would a senior on this team change it? No → skip.

## Output
| Sev | file:line | Issue | Trigger → impact | Fix |
Verdict: PASS (no CRITICAL/HIGH; zero rows is valid) | FAIL (list blocking rows).
Unread files: <name them, or "none">.
```

## 💥 Failure modes as rules
- **Reviewer as builder.** The reviewer never edits. A fix it applies itself is a fix nobody reviewed.
- **Same context reviews its own work.** Self-checks are weaker → [output-completeness.md](output-completeness.md#self-checks-useful-weaker-than-a-second-agent). Always use a separate agent.
- **Generic checklist pasted in.** React-hook rules on a SolidJS diff, Express advice on Hono. Write the rubric for the stack.
- **Severity by vibes.** CRITICAL means data loss, a security hole, or a prod outage on a reachable path. Nothing else qualifies.
- **Review as the gate.** Types, lint, and tests come first ([bin/check](../developer-experience/inner-loop.md)). The reviewer covers judgment, not syntax.
- **Hive reviews.** Each lens is a teammate with a disjoint question, not a disjoint file set. It's read-only, so it never conflicts → [hive-mind.md](../ai-agents/hive-mind.md).
