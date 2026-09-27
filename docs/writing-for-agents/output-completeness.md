# 🧾 Output Completeness — beating model laziness

**A partial output is a broken output.** Models truncate, emit placeholders, skip sections, and summarize instead of doing. Mostly a *behavior*, not a capability gap, so prompts, architecture, parameters and a mechanical gate all fix it. The gate is the one that sticks.

Provenance tags used below: **[upstream]** = distilled from published research / field reports (via the taste-skill research notes); **[ours]** = our practice. As of 2026-09.

## 🔎 What laziness looks like

| Shape | Example | Why it hurts |
|---|---|---|
| Placeholder in code | `// ... rest of code`, `// TODO: implement`, `pass  # add logic here`, bare `...` | compiles or parses, ships broken, nobody greps `TODO`s |
| Elision by reference | `// same as above`, `// similar for the remaining handlers` | the "remaining" never gets written |
| Skeleton for implementation | signatures + docstrings, no bodies | looks done in review at a glance |
| Middle skipped | first and last sections written, middle "follows the same pattern" | silent coverage hole |
| Describe instead of do | "you would add a retry wrapper that…" | zero executable output |
| Premature stop | "Let me know if you want me to continue" | turn ends at 40% |
| Undershot count | asked for 12 cases, got 7 | nobody counts |
| Silent truncation | output hits the token cap mid-function | half a file written to disk |

## 🧠 Root causes

| Cause | Mechanism | Tag |
|---|---|---|
| **Output-length ceiling** | Input windows are huge, output caps are far smaller. When the model predicts the full answer won't fit, it compresses *pre-emptively* instead of risking a hard cut. | [upstream] |
| **Brevity bias from post-training** | Alignment rewards short, confident answers; generation is the expensive part of serving. Learned preference: approximate > exhaustive. | [upstream] |
| **Stopping pressure** | Autoregressive models need a learned tendency to stop; tuned aggressively it drops required fields, ends with "want me to continue?". | [upstream] |
| **Effort shortcuts** | When a task looks easy or the context looks long, models reduce depth and return a surface summary — while still holding the information. | [upstream, "LazyBench", late 2024] |
| **Error avoidance** | Longer output = more surface for mistakes; shorter answers are "safer" for the model. | [upstream] |
| **Training-data bias** | Tutorials, forum answers and docs are full of `# implement here` and `...`. Placeholders read as *professional style*, not as omission. | [upstream] |
| **Middleware truncation** | Consumer chat apps cap history (reported ~32K tokens) and prune context before the model sees it. APIs and CLI agents don't. | [upstream] |
| **Our own caps** | `max_tokens` set low "to save money", step/turn caps, a tool result chopped to N chars. | [ours] — see [agent-work-limits](../ai-agents/agent-work-limits.md) |

## 🧪 Empirical findings

All **[upstream]**; numbers are from the cited studies on 2023–2025 models, **not** measurements on current Claude models. Treat as direction, not calibration.

| Finding | Source | Takeaway |
|---|---|---|
| No model met both length requirements and all sub-part instructions natively; mandatory sections omitted, lengths undershot | controlled study, 2025-12 (GPT-4 variants, DeepSeek) | multi-part asks need an explicit checklist |
| Truncated output matched the model's highest-confidence answer — no decoding failure | same study | it's a *choice*; sampling knobs won't cure it |
| Instructions/facts held over 200-turn conversations | same study | context loss is not the main cause |
| Stakes/incentive phrasing raised length and quality (reported +10% to +115%) | EmotionPrompt (Microsoft Research), 2023 | historically real; brittle; don't build on it |
| Shorter outputs in December; "It is May" in the system prompt lengthened them | "winter break" analysis, 2023-12 | arbitrary context shifts effort calibration |

**[ours]:** we don't use tip/"my career depends on it" framing. Current frontier models are tuned against it, it's unauditable, and a done-condition + a gate does the same job verifiably.

## ✍️ Prompt rules — paste-ready block

Drop into `CLAUDE.md`, a skill, or a subagent's system prompt. **[ours]**, adapted from the upstream output-enforcement skill.

```markdown
## Output completeness
- A partial output is a broken output. Full file asked → full file. N items asked → N items.
- FORBIDDEN in code: `// ...`, `// rest of code`, `// TODO`, `// implement here`,
  `// similar to above`, `/* ... */`, `pass  # implement`, bare `...` standing in for code,
  `throw new Error("not implemented")` in paths the task covers.
- FORBIDDEN in prose: "for brevity", "the rest follows the same pattern", "and so on"
  (replacing content), "let me know if you want me to continue", "left as an exercise".
- Never write a skeleton when an implementation was asked. Never describe code instead of writing it.
- Before starting: count the deliverables (files, functions, sections, cases). State the count.
- Before finishing: re-read the request, recount, add anything missing.
- Genuinely out of scope or blocked? Say so explicitly, name the missing piece, finish everything else.
- Long output: never compress the tail to fit. Stop at a clean boundary (end of function/file/section)
  and write: `[PAUSED — X of Y complete. Next: <name>]`. On "continue": resume exactly there, no recap.
```

Rules that make the block work:
- **Count, then recount.** A locked deliverable count is the cheapest completeness check that exists. [upstream]
- **Name the placeholder strings verbatim.** "Be thorough" binds nothing; a banned-token list does. [upstream]
- **Give a legal exit.** Without the `[PAUSED …]` escape, the model's only option near the cap is to compress. [upstream]
- **Honest blocker > fake completion.** Matches [behavioral-rules](behavioral-rules.md#-proactive--but-about-the-problem-not-the-diff): finish the rest, name what's left out.
- **Structure the prompt:** rules / context / data / numbered tasks in separate tagged blocks. Numbered tasks are countable; prose asks aren't. [upstream]

## 🏗️ Architectural patterns

Prompts lower the rate. Architecture removes the incentive: if no single call has to emit a huge answer, the model never pre-compresses.

| Pattern | How | When |
|---|---|---|
| **Outline → per-part → assemble** | call 1: structure only; call N: one component each, full; last call: integrate | output > a comfortable fraction of the cap [upstream] |
| **One file per call / per tool write** | agent writes each file via an edit/write tool, not one mega-message | code generation — the default in coding agents [ours] |
| **Edit, don't regenerate** | targeted edits on existing files instead of re-emitting the whole file | changes to large files; regenerating invites `// unchanged` elision [ours] |
| **Continuation loop** | detect the cap stop reason (`stop_reason: "max_tokens"` on the Claude API), feed back, ask to resume from the last clean boundary | any single long artifact [ours] |
| **Fan-out** | independent parts to parallel subagents, each owning a disjoint set of files | many similar units (handlers, migrations, docs) → [hive-mind](../ai-agents/hive-mind.md) |
| **Evidence first** | require tool output (test run, grep, fetched doc) *before* the narrative | analyses/reviews; stops "sounds right" summaries [upstream] |
| **Fresh-eyes finish reviewer** | separate agent, no edit rights, checks the deliverable against the original request, returns a verdict word + ordered fixes | anything user-facing or multi-part [ours, from the impeccable finish-reviewer idea] |
| **Grounding via tools/MCP** | fetch current docs instead of recalling them | models truncate or hedge when unsure of specifics [upstream] |

### Finish-reviewer rules (the "fresh eyes" pass)
- **Reviews the artifact, not the builder's summary.** The builder's "all done, implemented X/Y/Z" is not evidence.
- **Inventory from the request first**, then check the output against it — anchoring on the builder's plan inherits its omissions.
- **Fixed verdict vocabulary:** `ship` / `fix` / `rebuild`. Derived, not felt: any missing requirement → never `ship`.
- **Ordered fixes, most material first, capped** (e.g. ≤8). Missing deliverables outrank polish.
- **Re-check pass scores each prior fix** `resolved` / `partial` / `unresolved` from the artifact — narration of the fix doesn't count.
- **A "floor" list** of banned shapes (the placeholder list above) is checked even if a hook exists — hookless runs happen.

## 🎛️ Parameter tuning

| Knob | Rule | Tag |
|---|---|---|
| **Max output tokens** | Never lowball. Hitting the cap cuts mid-thought. Claude API as of 2026-09: current models allow up to 128K; default ~16K non-streaming, ~64K streaming; large values require streaming to avoid HTTP timeouts. | [ours, API docs 2026-09] |
| **Effort / reasoning depth** | Raise it for completeness-sensitive work. Claude (`output_config.effort`: `low`…`max`): `claude-opus-5-5` defaults to `medium` — set `high`/`xhigh` explicitly for coding and long agentic runs. | [ours, API docs 2026-09] |
| **Task budget** (where supported) | An *advisory* token budget the model can see lets it pace and finish gracefully instead of being cut by an invisible cap. Size from observed healthy max × margin. | [ours] → [context-budget](../ai-agents/context-budget.md#-tune-by-measurement--this-part-is-not-deterministic) |
| **Temperature / top-p** | Upstream advice: low temperature + low top-p for code. On current Claude models (Opus 4.7+, Opus 5.x, Fable 5.x) sampling params are **removed** (400). Don't rely on them. | [upstream] / [ours, 2026-09] |
| **Surface** | Consumer chat UIs truncate history; APIs and CLI agents don't. Real work runs on API/CLI. | [upstream] |
| **Model** | Prefer the latest model (e.g. `claude-opus-5-5`); older-model laziness numbers don't transfer. | [ours] |

**Read the terminator before touching a knob.** Log `stop_reason`/`finish_reason` + output token count per call. `max_tokens` → raise the cap or chunk. `end_turn` with placeholders → it's behavior: prompt + gate. [ours] → [agent-work-limits](../ai-agents/agent-work-limits.md#diagnose-the-terminator-before-you-touch-a-ceiling)

## 🛡️ The guard — reject placeholder output mechanically

Prose rules get ignored under pressure; a failing check doesn't. Climb the [guards ladder](guards-and-gotchas.md) — this is rung 4 (lint guard) + a hook. **[ours]**

### 1. Lint guard — scans the diff, not the whole repo
```bash
#!/usr/bin/env bash
# scripts/lint/no-placeholder-output.sh — fail on agent laziness markers in ADDED lines.
set -euo pipefail
BASE="${1:-origin/main}"
PATTERN='(//|#|/\*|<!--)\s*(\.\.\.|…)|rest of (the )?(code|file|implementation)|(implement|add) (this|logic|here|me)|similar(ly)? (to|for) (above|the rest|remaining)|same (as|pattern as) above|TODO:? implement|not implemented|omitted for brevity|for brevity|\bunchanged\b.*(code|remains)'
HITS=$(git diff --unified=0 "$BASE" -- . ':!*.md' ':!scripts/lint/no-placeholder-output*' \
  | grep -E '^\+[^+]' | grep -inE "$PATTERN" || true)
if [ -n "$HITS" ]; then
  echo "✖ placeholder output — write the real code, or name the gap in the PR body:"
  echo "$HITS"
  exit 1
fi
```

Guard rules (from [guards-and-gotchas](guards-and-gotchas.md#custom-lint-guards--institutional-memory-that-executes)):
- **Added lines only.** Legacy `TODO`s aren't this PR's problem; new ones are.
- **Test the guard:** a fixture with `// ... rest of code` must fail, a clean diff must pass.
- **Allowlist, don't weaken:** legit hits (a `...` spread operator, a test named `not implemented`) go in an explicit allowlist file.
- **Error text names the fix:** "write the real code, or name the gap in the PR body."
- **Also check for stubs:** new function bodies that are only `pass`, `throw new Error("TODO")`, `unimplemented!()`, `return null // TODO` — add per-language patterns as they bite.

### 2. Hook — catch it before the commit exists
Wire the same script into the agent harness so the agent learns in its own loop:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git commit:*)",
        "hooks": [{ "type": "command", "command": "scripts/lint/no-placeholder-output.sh HEAD" }]
      }
    ]
  }
}
```

Stronger variant: a `PostToolUse` hook on file writes/edits that greps just the written content and returns the hit to the agent immediately — the fix happens one step later, not at commit. Hook mechanics → [hooks-and-permissions](hooks-and-permissions.md).

### 3. Completeness check — the count gate
Placeholders are greppable; *missing* items aren't. For multi-part work:
- Plan lists deliverables as a checklist (files, endpoints, cases).
- Verification step diffs the checklist against reality: `ls`, `grep -c`, test names, route table.
- A deliverable with no evidence = not done. Report says what was **verified**, not assumed.

## 🚦 Scale it to the task

| Task | Do |
|---|---|
| One-liner, small edit | nothing extra — tool-based edits rarely elide |
| Single file / function | prompt block in `CLAUDE.md` + lint guard in CI |
| Multi-file feature | + deliverable count in the plan + hook on commit |
| Large generation (many files, long docs, migrations) | + outline→per-part chunking or fan-out + continuation handling + finish reviewer |

## ❌ Anti-patterns
- Low `max_tokens` "to save cost" — you pay twice: the truncated call and the retry. → [agent-work-limits](../ai-agents/agent-work-limits.md)
- Trusting "Done! I implemented everything" without a diff check.
- Bribes and threats in the prompt instead of a done-condition.
- Regenerating a whole large file to change ten lines.
- A `TODO` left as the record of a known gap — use a pinning test or a gotcha entry ([guards-and-gotchas](guards-and-gotchas.md#proceeding-anyway-leave-evidence)).
- Copying 2023-era model-specific numbers (length drops, % gains) into current tuning decisions.

---

**Related:** [behavioral-rules.md](behavioral-rules.md) — goal-driven execution · [guards-and-gotchas.md](guards-and-gotchas.md) · [hooks-and-permissions.md](hooks-and-permissions.md) · [../ai-agents/agent-work-limits.md](../ai-agents/agent-work-limits.md) · [../ai-agents/context-budget.md](../ai-agents/context-budget.md)

**Sources:** research notes and output-enforcement skill in [taste-skill](https://github.com/Leonxlnx/taste-skill); finish-reviewer idea from the impeccable skill's finish reviewer.
