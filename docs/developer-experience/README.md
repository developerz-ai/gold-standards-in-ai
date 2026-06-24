# 🛠️ Developer Experience

DX is not a nicety — it *is* agent speed. Every second shaved off the inner loop is shaved off every turn the agent runs. See [the philosophy](../00-philosophy.md).

| File | Topic |
|---|---|
| [dx-scripts.md](dx-scripts.md) | 🏃 `bin/setup`, `bin/dev`, `bin/check` — one command each |
| [dev-vps.md](dev-vps.md) | 🐧 Every dev on a Linux VPS with Claude Code — no OS wars |
| [codegraph.md](codegraph.md) | 🕸️ A knowledge graph of your code, for agents |
| [linting-ci.md](linting-ci.md) | ✨ Biome · 🔨 Blacksmith CI |

## The DX bar
A fresh clone is productive in **minutes**, not hours:
```bash
bin/setup     # prerequisites → install → .env → boot services → migrate
bin/dev       # full stack, hot reload
bin/check     # lint + typecheck + test (the gate)
```
Anything the agent needs that *isn't* one command is friction — turn it into one.
