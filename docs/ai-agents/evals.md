# 📏 Evals — measuring what the agent does

**Unit tests check your code. Evals check what the model does with your prompts, tools and skills.** Any change to those is a behavior change with no type error and no red test — only an eval sees it.

> Run evals **before merge** on any prompt/tool/skill/model change. Never in the default CI gate — they call live models, cost money, and are stochastic → [ai-first-cicd](../developer-experience/ai-first-cicd.md#evals-are-a-pre-merge-gate-never-a-ci-job).

## 🚦 When an eval is required

| Change | Suite |
|---|---|
| System prompt / `CLAUDE.md` block / skill body or description | routing + affected tasks |
| Tool added, renamed, collapsed, description edited | routing |
| Context-budget move (lazy surface, catalog trim) | routing — no worse than baseline → [context-budget](context-budget.md#-tune-by-measurement--this-part-is-not-deterministic) |
| Model ID bump (e.g. → `claude-sonnet-5-5`) | everything, per model |
| New agent capability | a new capability eval, written first |
| Pure code change behind a stable tool contract | none — unit tests cover it |

## 🧭 Two kinds — different questions, different bars

| | Capability eval | Regression eval |
|---|---|---|
| Asks | can it do this *at all* yet? | does it *still* do what it did? |
| Written | before building the feature (the spec) | when a capability ships, or after an incident |
| Expected pass rate | starts low, climbs | ~100% |
| Metric | pass@k | pass^k |
| Bar (as of 2026-10) | pass@3 ≥ 0.90 to ship | pass^3 = 1.00 on release-critical; no worse than baseline elsewhere |

A capability eval that reaches its bar **graduates** into the regression suite. Same case file, new metric.

## 📐 pass@k vs pass^k — report the one that matches the risk

- **pass@k** — at least one of k runs passes: `1 − (1 − p)^k`. "Can it, given retries?"
- **pass^k** — all k runs pass: `p^k`. "Will it, every time?"

| Single-run p | pass@3 | pass^3 | pass^5 |
|---|---|---|---|
| 0.70 | 0.97 | 0.34 | 0.17 |
| 0.90 | 0.999 | 0.73 | 0.59 |
| 0.95 | ~1.00 | 0.86 | 0.77 |
| 0.99 | ~1.00 | 0.97 | 0.95 |

**Tool routing, skill pickup and release-critical flows run on every turn — judge them on pass^k (k ≥ 3).** A 70% router looks fine on pass@3 and fails a user one turn in three. pass@k is for exploration: "is this capability reachable?"

Small k is a regression signal, not a reliability estimate. Say so in the PR.

## ⚖️ Graders — cheapest deterministic one that can decide

| Rung | Grader | Use for |
|---|---|---|
| 1 | **Code** — the tool-call trace, exit codes, `bun test` on the produced diff, file exists | routing, side effects, "did it build" |
| 2 | **Rule / schema** — Zod parse, regex, JSON shape, required fields | structured output, format contracts |
| 3 | **LLM judge** — rubric prompt on a stronger tier (`claude-opus-5-5`) | open-ended quality no rule can express |
| 4 | **Human** — label a sample | calibrating the judge; ambiguous cases |

- **Assert on the trace, not the reply.** Which tools were called, with which args, in what order. The reply's claims are not evidence → [output-completeness](../writing-for-agents/output-completeness.md#5-test-that-the-rules-actually-bind).
- **Judge rubrics are binary criteria**, not 1–5 scores: "cites the failing line — yes/no". The judge writes its reason *before* its verdict.
- **Calibrate the judge** against ~20 human-labeled cases; re-calibrate when its model or rubric changes. A judge that disagrees with humans is a flaky grader.
- **Flaky grader → fix it before it gates anything.** A gate that flips on identical input teaches everyone to re-run.

## 🗂️ Layout — evals are versioned with the code they test

```
evals/
├── routing/
│   ├── cases.jsonl              # {input, expect_tool, expect_args?, must_not_call?}
│   └── routing.eval.ts          # cheap, many runs per case
├── tasks/
│   └── refund-flow/
│       ├── task.md              # the prompt, as a user would write it
│       ├── fixture/             # seeded repo/db state
│       └── grade.ts             # code grader first; judge only for what code can't see
├── judges/
│   └── rubric-review-quality.md
└── baselines/
    └── claude-sonnet-5-5.json   # last accepted pass rates per suite, per model
```

```bash
bun run eval                          # all suites, k from config
bun run eval --suite routing --k 5    # one suite
bun run eval --baseline main          # diff pass rates vs the merge base
```

- Name files `*.eval.ts` so `bun test` never picks them up.
- Prompt, tool description and eval change **in the same PR**. An eval that lags the prompt tests nothing.
- `baselines/` updates only in a PR that explains the delta.

## 🎯 Cases — where they come from

- **Real transcripts and incidents first.** Every routing bug in prod becomes a case before the fix.
- **Negatives:** inputs where the right move is *no* tool, or the *other* similar tool. Routing evals with only positives reward always-calling.
- **Edge and adversarial:** ambiguous phrasing, missing args, injected instructions in tool results → [untrusted-input](untrusted-input.md).
- **Hold out a split.** Tune the prompt on one set, report on the other.
- A run that **stalls** (no progress — [agent-work-limits](agent-work-limits.md#the-one-bound-that-stays-lack-of-progress)) is a fail, never a skip.

## 💸 Cost per successful task

Report next to the pass rate: `total spend ÷ passing runs`. A cheaper model that needs twice the attempts or fails half the tasks is not cheaper. This is a measurement for choosing models and prompts — never a cap on the agent.

## 📋 PR paste format

```
Evals (k=3, base: main @ <sha>, model: claude-sonnet-5-5)
| suite    | metric | base | head | Δ     |
| routing  | pass^3 | 1.00 | 1.00 | 0     |
| refund   | pass@3 | 0.80 | 0.93 | +0.13 |
cost/success: $0.041 → $0.038
```

## ❌ Anti-patterns

| Don't | Why |
|---|---|
| Tune the prompt against the eval cases verbatim | overfit — passes the suite, fails users; keep a held-out split |
| Happy-path only | the router that always calls tool X scores 100% |
| Score only the final answer | right answer via the wrong tools is a routing bug waiting |
| LLM judge where a regex would do | slower, costlier, flakier |
| Run evals as a nightly/continuous job | nobody reads it; run them on the change that needs them, paste results in the PR |
| One run per case, report as reliability | n=1 is an anecdote |
| Chase pass rate, ignore cost and latency | a 2% gain at 3× cost is usually a loss |
| A/B the prompt in production instead | → [shipping-doctrine](../workflow/shipping-doctrine.md) — one path, measured before merge |

---

**Related:** [context-budget.md](context-budget.md) — evals gate every budget change · [tools-and-mcp.md](tools-and-mcp.md) — tool design the routing suite tests · [../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md) — where evals sit relative to the gate · [../writing-for-agents/output-completeness.md](../writing-for-agents/output-completeness.md) — behavior tests for prose rules
