# 🛠️ Harness Techniques — tokens saved, weak models finishing

Techniques coding-agent harnesses use for context, caching and tool calls, judged on **two axes at once**: tokens saved on a strong model, and whether a cheaper or smaller model can now **finish** the job. A technique that saves tokens and drops the weak tier's pass rate is a regression. Every adoption ships behind an eval run on at least one cheap model and the top one → [evals.md](evals.md).

Already covered, not repeated here: the standing-context budget, prompt-cache prefix order, lazy tool surfaces and chunked external results → [context-budget.md](context-budget.md); never capping the work → [agent-work-limits.md](agent-work-limits.md).

Provenance, as of 2026-11: **[measured]** = measured on one production agent fleet · **[practice]** = a gap or fix seen in production practice, no number attached · **[docs]** = vendor documentation read 2026-11 · **[unconfirmed]** = recalled from a public project or paper, primary source not re-fetched — direction only, no number to rely on.

## Meter first

Three per-model numbers decide most rows below. Without them the choice is taste.

| Metric | Decides |
|---|---|
| **cache-read share** of input tokens, per model | whether prefix work pays at all on that model — at 0% every step re-bills every byte |
| **edit-failure rate** (edit tool call rejected or mis-applied), per model | the edit-format policy |
| **tool-selection error rate** (wrong tool, malformed args), per model | how many tools a tier can see at once |

## The techniques, ranked

| # | Technique | Weak-model effect | Source |
|---|---|---|---|
| 1 | **Do not start doomed runs**: a relevance gate at claim ("is there anything to do?"), fast cancel when a newer event supersedes the run | none — it is pure waste removed | [measured] the largest single waste class was runs started with nothing to do, then runs cancelled after minutes of work |
| 2 | **Cache hit rate per model as a first-class metric** | **cheap providers cache by automatic prefix — only a byte-stable prefix helps them** | [measured]; [docs] provider caching docs: tools → system → messages order, a minimum cacheable length below which a mark is silently uncached |
| 3 | **Lint/typecheck after each edit batch**, diagnostics returned in the edit result | **catches a syntax or type slip one step later instead of forty** | [unconfirmed] an agent-harness ablation reported a lint-on-edit gain — re-fetch before citing a number |
| 4 | **Spill oversized tool output to a file**: size + path + preview in the result, the model greps the rest | **small windows survive big outputs**; no lost answers | [docs] at least one coding CLI spills large MCP results to a file instead of truncating |
| 5 | **CI logs to disk pre-chunked**, prompt orders "grep first, read second" | the failure line is found, not buried in 40k lines of passing output | [practice] |
| 6 | **Named failure counters with refunds**: an account limit or provider outage is never charged to the task's attempts | none directly — one outage stops burning every retry | [practice] |
| 7 | **Deferred tool schemas + client-side tool search** | **fewer tools in view → better tool choice**; gate on a tool-selection eval | [docs] tool search with deferred loading; [measured] a first-party tool block at tens of thousands of tokens per step vs under a thousand behind a gateway → [context-budget.md](context-budget.md) |
| 8 | **Read-before-edit as a tool precondition**, not a prompt line | the tool refuses an edit to an unread file; the model cannot skip it | [unconfirmed] a harness write-up reports prose versions of the rule are followed only part of the time; delete the prose once the tool enforces it |
| 9 | **Edit format per model tier**: exact unique-match-or-fail for strong models, a fuzzy apply ladder (exact → whitespace-insensitive → anchored) for weak ones | **the single biggest weak-model lever** — decide by measured edit-failure rate, not by model name | [unconfirmed] open-source coding agents ship fuzzy ladders — re-fetch their docs before copying one |
| 10 | **Dedupe the standing context** across layers (platform, role, repo brain, member prose) | a rule said three ways is three rules to a weak model | [practice] |
| 11 | **Agreement count across review samples**, carried into ranking and audit | free precision signal | [practice] |
| 12 | **Fan-out floor**: two members or none | no four agents for four one-line edits → [hive-mind.md](hive-mind.md) | [practice] |
| 13 | **Event-driven PR watch**, no polling | none — a scale limit removed | [practice] |
| 14 | **Replay fixtures**: one recorded production trajectory per incident, run through a mock model | orchestrator regressions caught at zero tokens | [practice] |
| 15 | **Non-progress fingerprint**: the identical tool call N times, no new state | the one bound that names a failure, not a bill → [agent-work-limits.md](agent-work-limits.md#the-one-bound-that-stays-lack-of-progress) | [practice] |
| 16 | **Deterministic before AI**: formatter, lint autofix, a re-run — before any model touches a red CI | zero tokens for the trivial class | [practice] |
| 17 | **Tool design for agents**: totals not raw payloads, a `concise`/`detailed` response format, errors that carry a corrected example, schema-validated input with a corrective retry | **an error that shows the fix is what a weak model needs** → [tools-and-mcp.md](tools-and-mcp.md) | [docs] vendor guidance on writing tools for agents |
| 18 | **External state re-pinned after compaction**: plan, todo, progress file re-inserted after every summary | **small-window models lean on files, not history** | [docs] vendor guidance on long-running harnesses |
| 19 | **Per-tier prompt variants**: same rules, fewer negations, explicit ordering, examples for every demanded format | the variant is chosen by measured capability class → [../writing-for-agents/prompts-for-weak-models.md](../writing-for-agents/prompts-for-weak-models.md) | [practice] |
| 20 | **Route by difficulty**: cheap models for search, summarise, classify; strong for plan and judge | a quality decision proven per tier on evals — never route by price alone | [docs] per-subagent model selection |

## Every clip leaves a door

Truncation is not the defect; **truncation with no way to reach the rest is.** Every bound on what the model reads picks one exit:

| Exit | Shape | Use when |
|---|---|---|
| Cursor / offset | a page plus how to ask for the next | naturally paged: search hits, a file, a log stream |
| Marker naming the door | the elision says what was dropped and **which tool** reaches it | one-shot values |
| Loud refusal | throw rather than answer from a partial | a verdict over the whole: a rollup, a count, a diff |

Keep the end that carries the answer:

| Artifact | Keep | Why |
|---|---|---|
| build / test / CI output | **tail** (head only to name what ran) | the failure prints last |
| compile / typecheck errors | **head** | the first error is the cause; the rest cascade |
| a diff | the anchor's neighbourhood | centre on the hunk under discussion |
| search hits, listings | head + a cursor | the order is the ranking |

A marker that says only "(truncated)" teaches the model to guess. Name the verb that fetches the rest.

## Declined — recorded so nobody re-litigates

| Technique | Why not |
|---|---|
| Embedding index over the repo | a code-structure index answers the same questions exactly → [../developer-experience/codegraph.md](../developer-experience/codegraph.md) |
| Precomputed background summaries | stale on the next commit; declined on evidence |
| A repo map in every prompt | the code index covers it on demand |
| Provider-side context editing as the only path | provider-specific; own-loop pruning stays the base, the provider feature an upgrade |

## Adopting a row

1. **Meter** the metric it moves, per model, with a date and the query.
2. **One row per PR**: a failing eval or reproducing test first, the change, the before/after on **two model tiers**.
3. A row that becomes a tool contract (8, 9, 15) **deletes the prose rule** it now enforces, in the same PR.
4. A provider-only feature (extended cache TTL, strict tool schemas, native structured output, deferred tool loading) ships behind a provider check that **degrades to the neutral path** — never the only path.
5. Before leaning on an **[unconfirmed]** row, fetch its primary source; drop the claim if it does not check out.

---

**Related:** [context-budget.md](context-budget.md) · [orchestration.md](orchestration.md) — model tiers · [../writing-for-agents/prompts-for-weak-models.md](../writing-for-agents/prompts-for-weak-models.md)
