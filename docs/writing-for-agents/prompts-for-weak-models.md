# 🪶 Prompts for Weak Models — same rules, less to reason about

A cheaper or smaller model follows the **same rules** as the top one; it fails on the **shape** they arrive in. Interlocking negations, exceptions twenty lines below their rule, a demanded format with no example, thirty tools in view — a frontier model absorbs those; a small one picks a wrong branch and keeps going. Rewrite the shape, never drop the rule.

Writing tool docs and recipes a weak model can execute is its own page → [../ai-agents/mcp-docs-for-agents.md](../ai-agents/mcp-docs-for-agents.md). Harness-side levers (edit format per tier, fewer tools, spill files) → [../ai-agents/harness-techniques.md](../ai-agents/harness-techniques.md).

## The rules

| Rule | Weak models fail when | Instead |
|---|---|---|
| **Rows, not prose loops** | a step list and a case list share one numbering; a first-match case hides an exception inside it | a table of `{trigger, condition, action, outcome}`; steps and cases in separate namespaces |
| **No step reads a later step's output** | step 2 scans a value only step 3 fetches | order by data flow; lint it if the prompt is generated |
| **Exception beside its rule** | "discard X" at line 49, "unless Y" at line 90 | one row carries the rule and its exception |
| **Positive over negative** | 25 lines of "never … unless … except" | say what to do; keep a "never" only for an irreversible act |
| **One term per thing** | `project member` here, `project-member` there, `maintainer` a third place | pick one token, use it everywhere — a small model treats variants as different things |
| **An example for every demanded format** | a strict output format with no example; one unparseable entry loses the whole answer | one complete, valid example inside the prompt, plus a schema the harness validates |
| **Explicit output contract** | "summarize your findings" | the exact fields, their types, the enum values, what "none" looks like |
| **Fewer tools in view** | thirty tools, three relevant to this phase | narrow the active set per phase; keep the discovery tool so nothing is unreachable → [../ai-agents/context-budget.md](../ai-agents/context-budget.md) |
| **Verification is a machine's** | "make sure the tests pass" with no command | the exact command and the exit code that means done → [../ai-agents/orchestration.md](../ai-agents/orchestration.md) |
| **Shorter, not vaguer** | a compact variant that drops the rule's reason | keep the one-clause *because*; cut the prose around it |

## Output contracts — make the parse unbreakable

```markdown
## Output
Return ONLY this JSON object. No prose before or after.

{"verdict": "approve" | "request_changes", "findings": [{"file": "src/a.ts", "line": 42, "rule": "N+1", "fix": "load ids in one query"}]}

- `findings` is `[]` when there are none — never omit it.
- `line` is an integer from the diff's new side.

Example (valid):
{"verdict": "request_changes", "findings": [{"file": "src/orders.ts", "line": 88, "rule": "N+1", "fix": "batch the per-order customer lookup"}]}
```

Harness side, always:

- **Validate against the schema**; on a failure, send the validator's error back **with the example** and retry once. An error that shows the fix is what a weak model needs.
- **Parse per entry where the shape allows** — one bad finding drops that finding, not the verdict.
- **Never accept prose where tokens were required**: a missing field is a refusal, not a default.

## Before / after

```markdown
# Before — 3 negations, an exception at a distance, an implicit order
Do not file an issue unless the CI failure is not flaky. Never comment on PRs
from bots except Dependabot. If the build failed, check the logs, but don't
re-run it if it already ran twice, unless the failure is an infrastructure error.
```

```markdown
# After — rows, one outcome each
| When | And | Do | Outcome |
|---|---|---|---|
| CI failed | the same test failed on the last 2 runs | file an issue with the test name and both run URLs | `filed` |
| CI failed | it passed on re-run | do nothing | `flaky` |
| CI failed | the log shows a runner/network error | re-run once | `rerun` |
| CI failed | it already ran twice and the log shows no runner/network error | do not re-run | `rerun_capped` |
| CI failed | the log is missing or shows no failure line | do not re-run; report the run URL and that the log had no evidence | `no_log_evidence` |
| PR author is a bot | author is not `dependabot[bot]` | do nothing | `skipped` |
```

## Choosing the variant

| Rule | Because |
|---|---|
| Select by **measured capability class** (eval pass rate, edit-failure rate, window size), never by model-name string matching | a name match breaks on the next release and silently sends the wrong variant to an unknown model |
| An unknown model gets the compact variant by default | the cheap BYOK long tail is exactly the set nobody hand-tuned |
| Same rules in both variants — **diff them in a test** | a compact variant that lost a rule is a policy fork |
| Prove each variant on evals at **two tiers** before merge | a rewrite that helps the small model can hurt the large one → [../ai-agents/evals.md](../ai-agents/evals.md) |
| Never A/B a prompt in production | a bug report cannot be reproduced without knowing which arm served it → [../workflow/shipping-doctrine.md](../workflow/shipping-doctrine.md) |

## Self-check

- [ ] Every conditional is a row with one outcome; no step depends on a later one.
- [ ] Every exception sits on its rule's row.
- [ ] Every demanded format has a complete valid example and a validating schema.
- [ ] One term per concept across the whole prompt.
- [ ] Only this phase's tools are active; discovery stays reachable.
- [ ] "Done" is a command and an exit code.
- [ ] The compact and full variants carry the same rule set (tested).

---

**Related:** [compressed-config.md](compressed-config.md) — dense prose for config files · [output-completeness.md](output-completeness.md) — a partial output is a broken output · [behavioral-rules.md](behavioral-rules.md)
