# 🧮 Context Budget & Prompt Caching

Everything your agent carries into a step — system prompt, tool schemas, skill catalog, injected config — is **re-paid on every step, used or not**. It is the one cost that scales with how *hard* the agent works. Treat the standing context as a budget with an owner, a number, and a test.

> Two rules, and they're both true at once:
> **Optimize the standing context, aggressively. Never trade capability for tokens.**
> The way both hold is **lazy**: good information, loaded on demand. Never a smaller catalog. Never a truncated result.

## The three standing costs

| Surface | Grows when | Fix |
|---|---|---|
| **Tool schemas** | every feature adds a tool | fewer tools, more parameters; move connectors behind a constant-size gateway |
| **System prompt** | every incident adds a paragraph | rules only the model can't infer; move the rest into skills |
| **Skill catalog** | every domain adds a skill | one line per skill, body loaded on demand |

Measure them, don't estimate them. A measured example (2026): a first-party tool block at **~38k tokens per step**, versus **~800 tokens** for a gateway exposing a 47-tool connector lazily. That is a 47× difference paid on every single step of every turn.

**Guard the numbers with tests, and generate the inventory:**

```ts
// test/unit/tool-schema-budget.test.ts
it("standing tool surface stays under budget", () => {
  expect(serializedToolBytes()).toBeLessThan(TOOL_SCHEMA_BUDGET_BYTES);
});
```

```bash
bun scripts/docs/gen-tool-surface-inventory.ts --check   # committed doc must match reality
```

Never hand-copy a byte count into a doc — every copied instance is wrong within days. Generate it, `--check` it in lint, and let prose *point at* the live number.

## 💾 Order the prompt by what changes, not by what reads well

Providers cache a **matching prefix**. One variable value placed early invalidates every cached byte after it. So prompt order is a cache decision before it is a token decision:

```
[ static platform prose ]        ← never changes → cached for everyone
[ org / space configuration ]    ← changes rarely
[ per-turn state, timestamps ]   ← changes every turn → put it LAST
```

- A timestamp, counter, or freshly-serialized map near the top is **the single most expensive one-line mistake** available in an agent codebase.
- Serialize maps/sets deterministically (sorted keys) — non-deterministic ordering is a silent cache-buster.
- **Pin prefix stability with a test** that asserts the first N bytes are byte-identical across two builds with different per-turn state.
- Check your **cache hit rate per model** before optimizing anything else. A model at 0% hits re-bills every byte at full input rates on every step — there, the standing context *is* the bill.
- Send a stable cache key per conversation/tenant where the provider supports it.

## 🎣 Lazy on both surfaces

**What you author** — tools, skills, prompts. Yours to shrink: list less, keep everything reachable by name or search.

- Discovery pattern: `discover → describe → dispatch → fetch`. The surface is constant; the catalog is not in context.
- The property that matters is **"none of the tools is per-connector"**, not the raw tool count.
- Skills: a one-line catalog entry + `load_skill(name)`. Pin a body inline only when it's needed on *most* turns.
- Prefer **fewer tools with more parameters** over a tool per verb → [tools-and-mcp](tools-and-mcp.md#designing-your-own-mcp-server--the-3-tool-pattern).

**What comes back from outside** — MCP results, HTTP responses, data-source reads. **You don't control these payloads.** One connector call has been measured returning **28,181 bytes** into a single step, permanently polluting the turn.

The rule: **a small external result passes through unchanged; a large one gets split, and the agent pulls the chunks it needs.** Copy `web_fetch`'s shape — return a handle plus enough structure to navigate:

```json
{ "handle": "res_01H…", "bytes": 28181, "shape": { "rows": 412, "fields": ["id","name","balance_usd","updated_at"] },
  "preview": [ { "id": "…", "name": "…" } ],
  "next": "fetch_chunk(handle, offset=50, fields=['id','balance_usd'])" }
```

- **Never truncate silently.** Every byte stays reachable, one chunk at a time.
- **"It was too big" must never become a missing answer** — that's a capability regression wearing a cost-saving costume.

## 🧰 Writing lazy tools (Vercel AI SDK)

The tools you hand the agent are where laziness is actually implemented. Four patterns, all in plain `tool()` definitions:

**1. Constant-size gateway instead of per-connector tools.** Three tools cover any number of backends; the catalog lives *behind* them.

```ts
import { tool } from "ai";
import { z } from "zod";

export const discover = tool({
  description: "List available connectors/resources and their actions. Call before describe/dispatch.",
  inputSchema: z.object({ query: z.string().optional() }), // `{}`-callable: no required args
  execute: ({ query }) => registry.catalog(query),          // one line per resource + action names
});

export const describe = tool({
  description: "Full input schema for one resource+action. Accepts an array to batch.",
  inputSchema: z.object({ resources: z.array(z.string()).min(1) }),
  execute: ({ resources }) => registry.schemas(resources),  // cached per session
});

export const dispatch = tool({
  description: "Call a resource action with its params.",
  inputSchema: z.object({ resource: z.string(), action: z.string(), params: z.record(z.unknown()) }),
  execute: (i) => registry.run(i),
});
```

**2. Make read verbs callable with `{}`.** A required argument the model can't guess turns a one-step read into a describe round-trip. Default sensible values server-side.

**3. Return handles for big payloads, never a truncated blob.**

```ts
execute: async (args) => {
  const rows = await source.read(args);
  const body = JSON.stringify(rows);
  if (body.length <= INLINE_LIMIT) return rows;                 // small → pass through unchanged
  const handle = await blobs.put(body);                          // large → split, stay reachable
  return { handle, bytes: body.length, rows: rows.length,
           fields: Object.keys(rows[0] ?? {}), preview: rows.slice(0, 3),
           next: `fetch_chunk({ handle, offset: 3, fields: [...] })` };
}
```

**4. Load knowledge on demand, never into the prompt.** Memory, skills, and docs are tools, not prompt sections: `search_memory(query)`, `load_skill(name)`, `read_doc(path)`. A one-line catalog entry is enough for the model to know a skill exists; the body arrives only when it's picked.

Wire-up rules:
- **Cache MCP `tools()` per session** — re-listing on every step is both slow and expensive.
- **`prepareStep` / dynamic tool sets:** narrow the *active* tool set per phase when the phase is unambiguous, but keep the discovery tool present so nothing becomes unreachable.
- **Descriptions are the routing signal.** They're the part you cannot cut — trim schemas and examples first, never the sentence that tells the model when to reach for the tool.
- **Every collapse is eval-gated.** If tool-selection accuracy drops, the byte saving doesn't count.

## 🚫 What "never trade capability for tokens" forbids

Reducing the agent's *work* is always wrong. Reducing the *bytes it carries to do that work* is always right.

| ❌ Not allowed | ✅ Allowed |
|---|---|
| Capping steps or tool calls | Making the tool block lazy |
| Trimming context the agent needs | Reordering the prompt for cache hits |
| Dropping catalog entries | One-lining catalog entries, bodies on demand |
| Truncating a tool result | Chunking it behind a handle |
| Collapsing tools in a way that hurts routing | Collapsing tools that provably don't hurt routing |

The test is not "is the block smaller" but **"can the agent still find and use the thing."** Which means the gate is an eval, not a byte count.

## 📊 Tune by measurement — this part is not deterministic

Thresholds, budgets, how many catalog lines to render, what to inline vs load on demand: **none of these has a provably-correct value**, and a green typecheck tells you nothing. House rules:

1. **Every such number carries its derivation in a comment** — what was observed, when, and the multiple applied.
2. **Size from an observed healthy MAXIMUM × a comfortable multiple**, never from an average.
3. **Gate the change on evals** — a tool-selection eval and a skill-pickup eval. A collapse or a lazy surface ships only if those are *no worse than baseline*.
4. **Re-measure before changing one.** A drifted number is worse than an untuned one, because it still looks deliberate.
5. **Ship one path** — never A/B a prompt in production: each arm doubles the untested combinations, and a bug report can't be reproduced without knowing which arm served it → [../workflow/shipping-doctrine.md](../workflow/shipping-doctrine.md).

```ts
// Derivation lives next to the number, or the number is folklore.
/** Observed healthy max across 30 days of prod turns: 41 steps (p100).
 *  Backstop = 41 × 1.5 ≈ 60. Re-measure before touching. Not a work budget. */
export const RUNAWAY_STEP_BACKSTOP = 60;
```

---

**Related:** [agent-work-limits.md](agent-work-limits.md) — why a *step* budget is a different thing from a token budget · [tools-and-mcp.md](tools-and-mcp.md) · [../writing-for-agents/compressed-config.md](../writing-for-agents/compressed-config.md) — the same economics, applied to the files *your* coding agent reads
