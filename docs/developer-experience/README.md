# 🛠️ Developer Experience — and the "developer" is the AI

**Only the AI writes the code.** So DX is not a nicety for humans; it is the **agent's runtime**. Every second, every guess, every undocumented step is paid by the thing doing all the work — on every turn, forever. Optimize it the way you'd optimize a hot loop. See [the philosophy](../00-philosophy.md).

| File | Topic |
|---|---|
| [dx-scripts.md](dx-scripts.md) | 🏃 `bin/setup`, `bin/dev`, `bin/check` — one command each |
| [inner-loop.md](inner-loop.md) | ⏱️ Scoped checks vs the full gate — the loop the agent actually runs |
| [dev-vps.md](dev-vps.md) | 🐧 Every dev on a Linux VPS with Claude Code — no OS wars |
| [codegraph.md](codegraph.md) | 🕸️ A knowledge graph of your code, for agents |
| [linting-ci.md](linting-ci.md) | ✨ Biome · 🔨 Blacksmith CI |
| [ai-first-cicd.md](ai-first-cicd.md) | 🤖 CI as the agent's feedback channel — one gate, parity, agent-readable failures |

## The DX bar
A fresh clone is productive in **minutes**, not hours:
```bash
bin/setup     # prerequisites → install → .env → boot services → migrate
bin/dev       # full stack, hot reload
bin/check     # lint + typecheck + test (the gate)
```
Anything the agent needs that *isn't* one command is friction — turn it into one.

## Make it easy, or the agent routes around it
An agent doesn't complain about a bad workflow; it silently invents a worse one. Hard things get worked around, and the workaround becomes the convention.

| Friction | What the agent does instead | Fix |
|---|---|---|
| Slow whole-repo gate | pushes without running it | a fast [scoped tier](inner-loop.md) |
| Env setup needs tribal knowledge | hand-writes a `DATABASE_URL` that half-works | self-provisioning harness + `bin/dev doctor` |
| Deploy status is "ask someone" | claims a success it never verified | a live probe command |
| A capability exists but isn't listed | invents a workaround, or calls it impossible | keep it reachable by name/search → [lazy](#lazy-by-default) |

**Rule:** the second time something is awkward, it becomes a command → [guards & gotchas](../writing-for-agents/guards-and-gotchas.md#the-threshold--when-to-automate).

## Lazy by default
What the agent reads *every turn* is expensive; what it can fetch *on demand* is free until needed. Keep the always-on layer tiny and make depth discoverable:

- `CLAUDE.md` is a router, not an encyclopedia — deep docs live in `docs/` and load when relevant.
- Skills: one catalog line, body on demand.
- Scripts: a `bun scripts/help.ts` catalog, not a list in `CLAUDE.md`.
- Tool surfaces: discover → describe → dispatch, never a per-connector tool list → [context budget](../ai-agents/context-budget.md).

**Lazy never means less.** Never a smaller catalog, never a truncated result — only *later*.

## Be proactive
The agent is not a ticket-taker. Inside the work it's already doing:

- **Fix the bug you see** in a file you're already editing — and say you did.
- **Automate the ritual you just performed by hand**, in the same PR.
- **Ship the guard with the fix** so the defect class can't come back.
- **Propose the feature the code is asking for**; let the human decide scope.
- **Say "this shouldn't be built"** when nothing breaks by doing nothing. Recovering the real problem beats delivering a solution to the wrong one.
