# 🧠 Philosophy: AI-First Development

The whole system rests on one loop:

> **Great DX → fast agents → more shipped → better DX.**

Everything in this repo is in service of making that loop spin faster. An AI agent is a teammate that works at the speed of your tooling. Slow tests, hidden commands, undocumented conventions, and OS quirks each tax *every single turn*. Remove them once and the agent compounds.

## Who this is for: AI writes ~100% of the code
These standards assume an **AI-first org where agents write essentially all the code** and humans review, steer, and operate. The whole game is making each repo so legible and well-tooled that an agent ships production-quality work autonomously — and a human verifies it at the PR, not by typing the code.

The human-in-the-loop points that stay human (or human-supervised):
- **Review & merge** — every PR is reviewed (by a person and an [automated AI reviewer](developer-experience/linting-ci.md)) before it lands.
- **Deployments** — handled by a deployment agent (we call ours **Ana**), **most of the time under human supervision**. The agent runs the deploy; a human watches the rollout and can stop it.
- **Direction** — what to build and why stays a human call.

If humans still write most of your code, most of this still helps — but the rules are tuned for full autonomy.

### The quality net: defense in depth
A coding agent is fast but fallible — it *will* miss things. So quality doesn't rest on the agent being perfect; it rests on layers that catch what it misses:

1. **Tests** ensure high quality — they prove the change does what it should and catch regressions. The agent writes them and runs them every loop. [→ testing](architecture/testing.md)
2. **Automated AI review** on every PR catches what the coding agent missed — bugs, security issues, convention drift — at machine speed. [→ AI review](developer-experience/linting-ci.md)
3. **Human review + supervised deploy** is the final gate.

Each layer assumes the one before it is imperfect. That's what makes shipping at agent speed *safe*.

## The seven principles

### 1. 🏎️ Great DX *is* agent speed — and the "developer" is the AI
Only the AI writes the code, so **DX is the agent's runtime**, not a human comfort. An agent's effectiveness is proportional to iteration speed. A test that takes 60s instead of 5s doesn't slow you down once — it slows down every loop the agent runs, forever. Optimize the inner loop ruthlessly:

| What | Target |
|---|---|
| Unit tests | < 10s |
| Lint | < 5s |
| Single test file | < 3s |
| Dev server start | < 5s |
| Full CI (push → green) | < 5 min |

And **easy beats documented**: an agent doesn't complain about a bad workflow, it silently invents a worse one. Anything awkward the second time becomes a command. See [developer-experience/](developer-experience/README.md) · [the inner loop](developer-experience/inner-loop.md).

### 2. 🔓 Freedom and access — the agent must be able to do the work
Two halves of the same principle: **give it the keys, then don't cut it off mid-thought.**

**Access.** An agent that can read prod (read-only), trigger a build, query the DB, open a browser, and check error logs solves problems end-to-end. One that has to ask you to do each step is a glorified autocomplete.

- Wrap every external system in a `scripts/` command the agent can run. [→ DX scripts](developer-experience/dx-scripts.md)
- Commit a `.mcp.json` so every agent on the repo gets the same reach with zero setup. [→ .mcp.json](writing-for-agents/mcp-json.md)
- Connect MCP servers for issues, browsers, errors, DB access. [→ tools & MCP](ai-agents/tools-and-mcp.md)
- Build a [CodeGraph](developer-experience/codegraph.md) index so it knows the codebase structurally.
- Run with broad permissions + quality-gate hooks instead of approval-prompting every action. [→ hooks & permissions](writing-for-agents/hooks-and-permissions.md)

Access is gated by *safety in the tools* (read-only DB roles, audited gateways, reviewed PRs), not by withholding capability. **A missing capability is a bug in your setup**, not something to tell the agent to work around.

**Freedom.** No step cap, no tool-call cap, no per-turn token or cost cap. Value produced > price of tokens; a limit doesn't make the agent cheaper, it makes it dumber — the human finishes the half-done work anyway. Bound **lack of progress**, never honest work; an agent that can't finish stops itself and names what it's missing. [→ agent work limits](ai-agents/agent-work-limits.md)

### 3. 🐧 Every dev on a Linux VPS with Claude Code
No more "works on my machine." Each developer gets a Linux VPS (or dedicated box) with Claude Code installed and the workspace cloned. **One OS, one setup, one set of commands.**

- Kills the Linux/Windows/Mac fragmentation tax — scripts, paths, and tooling behave identically for everyone.
- Agents run in the same environment they'll deploy to.
- Onboarding is one command. [→ dev VPS standard](developer-experience/dev-vps.md)

### 4. 📦 Monorepo so each part uses the best language for the job
A monorepo lets one product mix runtimes without friction: SolidJS + Bun for the web, Rust for the hot-path API, Swift/Kotlin for native mobile, a C++ core where it matters — all sharing one `CLAUDE.md`, one CI, one set of conventions. Pick the right tool per component; the agent sees the whole picture. [→ monorepo](architecture/monorepo.md)

### 5. 📝 Write everything down — for the next session, not for posterity
Agents are stateless across sessions except for what you write down. A `CLAUDE.md` that documents the exact commands and conventions is worth more than any amount of clever prompting. Compress it (it's read every turn), date load-bearing claims, and delete what rots. [→ writing for agents](writing-for-agents/README.md)

Better than writing it down: **make the machine careful.** A rule in prose is advice; a lint guard that fails CI is a fact the next agent cannot miss. [→ guards & gotchas](writing-for-agents/guards-and-gotchas.md)

### 6. 🎣 Lazy by default — good information, loaded on demand
Everything read *every turn* is paid forever; everything fetchable *on demand* is free until needed. Keep the always-on layer tiny — `CLAUDE.md` as a router, one-line skill catalogs, a constant-size tool surface — and make depth discoverable by name or search.

**Lazy never means less.** Never a smaller catalog, never a truncated result — only *later*. A capability that's merely unlisted is still reachable; a capability that's hidden reads as impossible and the agent invents a workaround. [→ context budget](ai-agents/context-budget.md)

### 7. 🚀 Be proactive — solve the problem, not the ticket
An agent that only does exactly what was typed wastes the largest part of its value. Inside the work it's already doing: fix the bug it sees in the file it's editing, ship the guard with the fix, automate the ritual it just did by hand — **in the same PR** — and propose the feature the code is asking for.

The counterweight: **recover the problem before building the solution.** "Nothing breaks by doing nothing" and "this already exists here" are successful outcomes. Proactive means owning the outcome, not widening the diff. [→ behavioral rules](writing-for-agents/behavioral-rules.md)

## The setup that makes an agent excellent

A repo is "AI-first" when a fresh agent can, with zero hand-holding:

1. Read `CLAUDE.md` → know the stack, structure, and exact commands.
2. Run `bin/setup` → working environment in minutes.
3. Run `bin/dev` / `bin/check` → boot the stack, run the gate.
4. Find any symbol via [CodeGraph](developer-experience/codegraph.md) or a predictable file layout.
5. Make a change, run the relevant test, and know it's green.
6. Commit through a hook that auto-lints, push, open a PR.

If any step requires tribal knowledge, that's a bug in your DX — fix it in the repo, not in the prompt.

## ⚡ Fail fast · fix fast · deploy fast
Velocity comes from a tight error loop, not from cutting corners:
- **Fail fast** — surface errors loudly and early. A crash on a bad assumption beats silent wrong behavior. Don't swallow exceptions; don't retry a fatal error.
- **Fix fast** — production errors are a *work queue*. [Sentry](infrastructure/observability.md) → an agent reads the trace, writes a fix + a reproducing test, opens a PR.
- **Deploy fast** — small PRs, fast cached CI, GitOps that ships a merged change in minutes. Deploys are agent-run, [human-supervised](#who-this-is-for-ai-writes-100-of-the-code).

Small change → fast feedback → fast fix → fast ship. The smaller and faster each step, the more an agent can own the whole loop.

## 🛡️ Very low undefined behavior
An autonomous agent can't be debugging mystery states. Engineer the system so behavior is predictable:
- **Type-safe everywhere** — TypeScript strict (no `any`), Zod at every boundary, Rust's type system. Make illegal states unrepresentable.
- **Custom, typed errors** — never a bare `Error`/`Exception`. No `unwrap` on a hot path.
- **Exhaustive handling** — discriminated unions + exhaustive switches; handle every case or fail loudly.
- **Determinism** — no hidden globals, no implicit state, idempotent scripts and migrations.

Low undefined behavior is what makes "fail fast" safe: when something breaks, *where* and *why* are obvious.

## Coding values (non-negotiable)
- **SOLID, especially SRP.** One file, one job. Files ≤ ~500 LOC. [→ SOLID/SRP](architecture/solid-srp.md)
- **Tests always.** They're how the agent verifies its own work. [→ testing](architecture/testing.md)
- **Custom error types**, not generic exceptions.
- **Type-safe everything** — TypeScript + Zod, Rust's type system, strict configs.
- **Surgical diffs.** Every changed line traces to the task. [→ behavioral rules](writing-for-agents/behavioral-rules.md)
- **One path, replaced completely.** No shims, no dual read/write, no feature flags, no prod A/B. [→ shipping doctrine](workflow/shipping-doctrine.md)
- **Shape now, capacity later.** Keyset, bounds, idempotency up front; sharding and rollups behind a written trip condition. [→ data & scale](architecture/data-and-scale.md)

---

Next: [✍️ Writing for agents](writing-for-agents/README.md) — the highest-leverage section.
