# 🧬 The Agent SDK — Loop, Tools, Output

The foundation of every agent: a loop that calls tools until it's done. Examples use the Vercel AI SDK (`ai`); the same shapes apply to the Claude Agent SDK.

## The loop
Single-turn with tools:
```ts
import { generateText, tool } from "ai";
import { z } from "zod";

const result = await generateText({
  model,
  system: "You are a senior developer. Make the change, run tests, stop.",
  prompt: task,
  tools: { readFile, writeFile, bash, grep },
  maxRetries: 5,
  abortSignal: controller.signal,
});
// result.text · result.steps · result.totalUsage
```
> ⚠️ Use `result.totalUsage` (summed across all steps), **not** `result.usage` (last step only) — or you miscount tokens and cost.

Multi-turn agent (loops until a stop condition):
```ts
import { ToolLoopAgent, hasToolCall } from "ai";

const agent = new ToolLoopAgent({
  model, tools, instructions,
  stopWhen: hasToolCall("submit"), // no step cap — stall detection ends a stuck run
  onStepFinish: async ({ text, toolCalls, usage }) => { /* stream + log */ },
  prepareStep: async ({ messages }) => ({}), // compact context / inject guidance mid-run
});
const out = await agent.generate({ prompt, abortSignal: controller.signal });
```

## Tools — Zod schemas
```ts
const readFile = tool({
  description: "Read a file by path",
  inputSchema: z.object({ path: z.string(), limit: z.number().optional() }),
  execute: async ({ path, limit = 200 }) => ({ content: await read(path, limit) }),
});
```

Collapse related ops into one tool with a discriminated union:
```ts
const git = tool({
  description: "Git ops (status|diff|commit)",
  inputSchema: z.discriminatedUnion("action", [
    z.object({ action: z.literal("status") }),
    z.object({ action: z.literal("diff"), args: z.string().optional() }),
    z.object({ action: z.literal("commit"), message: z.string(), files: z.array(z.string()) }),
  ]),
  execute: async (a) => runGit(a),
});
```

**Action allowlist** = capability tiers from one tool factory:
```ts
const readOnly = { file: fileTool(["read","search","grep"]), git: gitTool(["status","diff","log"]) };
const coding   = { file: fileTool(["read","write","edit","search"]), git: gitTool(["status","diff","commit"]), bash };
```
Parallel tool calls happen automatically — the model can request several in one step; results return in order.

## Structured output
Force a schema via a terminal `submit` tool (best for sub-agents):
```ts
const Review = z.object({
  issues: z.array(z.object({ severity: z.enum(["critical","warning","info"]), location: z.string(), description: z.string() })),
  summary: z.string(),
});
const submit = tool({ inputSchema: Review, execute: async (o) => o });
// after run: find the `submit` tool call, Review.parse(call.input)
```

## Streaming, retries, cost control
- **Stream** via `onStepFinish` — push text deltas and tool calls to the client live; `abortController.abort()` to cancel.
- **Retry middleware:** retry transient errors (429, 500/502/503/504, timeout) with exponential backoff; **fail fast** on fatal ones (402 / insufficient credits / auth). Never retry a credits-exhausted error.
- **Stop conditions:** stop on *non-progress* — no new tool signature or text, same tool call with the same args repeating, no state change — never on a step/tool-call count → [stall rule](agent-work-limits.md#the-one-bound-that-stays-lack-of-progress).
- **Context window:** never truncate. Keep the standing context lazy, chunk big tool results behind a handle, compact history (below) → [context-budget](context-budget.md).

## Parser first, LLM for the uncertain
Structured, repeating text (logs, invoices, quiz banks, config dumps) → a deterministic parser does the bulk; the LLM sees only what the parser isn't sure of.
```
text ─▶ parser/regex ─▶ items + confidence ─┬─ ≥ threshold ─▶ output
                                            └─ < threshold ─▶ LLM ─▶ output
```
- Every item carries a **confidence** (all fields matched, counts tie out, no leftover text).
- **Threshold set by an eval**, not by feel: labelled sample, measure accuracy per band, move the line → [evals](evals.md).
- Free-form, non-repeating text → straight to the LLM; a parser there is wasted work.

## Memory across turns
- Persist messages to a DB (role, content, tool calls, token usage) for replay.
- **Compact** when history exceeds a threshold: summarize the oldest messages, keep the most recent N intact. The summary **always keeps the tried/failed list** — approaches attempted, why each failed — or the agent re-tries them after compaction.
- **Inject guidance mid-run** via `prepareStep` — append a `[Human guidance]` user message before the next step.

## System prompt structure
Role → what it has access to → rules → "do NOT" → current task → repo context. Load repo context from `CLAUDE.md` + `README` + `.claude/` (cap the total). Task-specific role lines (implement vs bug-fix vs triage vs refactor) sharpen behavior. Same principles as [writing for agents](../writing-for-agents/README.md) — you're writing a system prompt the same way you'd write a `CLAUDE.md`.

Next: compose loops into [orchestration](orchestration.md).
