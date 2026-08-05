# 🔓 Give the Agent Freedom and Access

An agent produces value in proportion to what it is **allowed to do** and what it can **reach**. Most "the AI isn't good enough" reports are actually one of two self-inflicted wounds: a limit that stopped it mid-thought, or a missing capability it couldn't route around.

> **Value produced > price of tokens.** A limit does not make the agent cheaper — it makes it dumber. The human finishes the half-done work anyway, and stops trusting the tool.

Two halves, and you need both:

| | Freedom | Access |
|---|---|---|
| What it means | no cap on how much work one turn may do | every system it needs, reachable as a tool |
| Failure if missing | half-finished answers, silent truncation | "I can't check that" → guessing |
| Where it's decided | the agent loop | the tool surface + permissions |

## 🚫 Freedom: never cap the work

**Not a tradeoff to weigh — settled.** No step cap, no tool-call cap, no per-turn token cap, no per-turn cost cap, no duration cap. Solving the problem beats saving tokens; an agent doing *more* work is the feature, not the cost.

For the **attended** lane — a human is waiting on that turn — make it a compile error to reintroduce one:

```ts
export const ATTENDED_TURN_WORK_BOUNDS = {
  maxSteps: undefined,      // typed `undefined` on purpose:
  maxToolCalls: undefined,  // adding a number here does not compile.
  maxCostUsd: undefined,
} satisfies WorkBounds;
```

An explicitly authorized, operator-set cap wins in either lane. **An explicit config value is never silently reduced** — honor it, or throw. (Real incident: a platform ceiling quietly rewrote an org's deliberate `1_000_000_000` budget down to `5_000_000` every turn and only `console.warn`ed.)

### The one bound that stays: lack of progress
A turn dies when it **stops getting anywhere**, never because honest work took a while. Fence on *stall*, not on volume — no new information, no state change, repeated identical calls.

And an agent that cannot finish **stops itself and names what it is missing** — missing access, missing tool, ill-posed task. That is a successful outcome. Grinding, guessing, or fabricating an answer is not.

### Unattended runs keep a backstop — which is the same rule, not an exception
Nobody is watching a background run, so it gets a **runaway backstop**: sized from an observed healthy **maximum × a margin**, documented with its derivation, and far above the deepest healthy run. Never a work budget. Never sized from an average. → [context-budget](context-budget.md#-tune-by-measurement--this-part-is-not-deterministic)

### Diagnose the terminator before you touch a ceiling
Every visible number can point at a wall the agent never hit. A real production turn died after **2 steps of 60, 2 tool calls of 100, $0.053 of $4.00** — no ceiling was involved. The terminator was an upstream error: `finish_reason=error`, **0 tokens in and out**, i.e. the call failed before the model saw it.

Instrument for this: persist `finish_reason`, token counts, and step spans per turn, and read them **first**. A correlation in time is not a mechanism — the first, very convincing hypothesis in that incident ("a deploy rolled the single-replica gateway") was falsified by the rollout config.

## 🔑 Access: wire it to everything, safely

The agent that can read prod (read-only), query the DB, drive a browser, read the error tracker, trigger a build, and call your own admin API solves problems end to end. The one that must ask a human for each step is autocomplete with extra latency.

| Give it | Via | Guardrail that makes it safe |
|---|---|---|
| Real data | DB gateway MCP | read-only role, per-grant scoping, synchronous audit |
| Production truth | error monitor, log/metric queries | read-only tokens |
| The UI | headless browser MCP | test/staging creds; an AI-debug escape hatch for captcha |
| Your product itself | your own admin/first-party MCP | scoped tokens, approval gate on writes |
| The codebase's structure | [CodeGraph](../developer-experience/codegraph.md) | read-only by nature |
| Every external system | a `scripts/<domain>/<verb>.ts` wrapper | mutations require `--write` and print first |

Rules that keep "maximum access" and "no leaked credentials" true at the same time:

- **Safety lives in the tool, not in withholding the tool.** Read-only roles, audited gateways, `--write`-gated scripts, reviewed PRs.
- **Credentials never reach the agent.** It sends a request + an identity token to a gateway; the gateway holds the secret → [tools-and-mcp](tools-and-mcp.md#audited-capability-access-the-gateway-pattern).
- **Approve classes, not calls.** Broad allow-lists for reads; a human gate only for genuinely irreversible writes → [../writing-for-agents/hooks-and-permissions.md](../writing-for-agents/hooks-and-permissions.md).
- **A missing capability is a bug in your setup**, filed against the repo — not a thing to tell the agent to work around.
- **Hidden capability is worse than missing capability**: a tool the agent can't discover reads as "impossible" and it invents a workaround. Keep everything *reachable by name or search* even when it's not listed → [context-budget](context-budget.md#-lazy-on-both-surfaces).

## The pairing

Freedom without access = an agent free to spin on a question it can't answer.
Access without freedom = an agent that reaches the right data and gets cut off before it can use it.

Ship both, and the remaining failure mode is an honest one: the agent says what it's missing, and you fix the environment.

---

**Related:** [context-budget.md](context-budget.md) — shrink the *bytes*, never the work · [orchestration.md](orchestration.md) · [../00-philosophy.md](../00-philosophy.md#2--freedom-and-access--the-agent-must-be-able-to-do-the-work)
