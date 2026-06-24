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

## The five principles

### 1. 🏎️ Great DX *is* agent speed
An agent's effectiveness is proportional to iteration speed. A test that takes 60s instead of 5s doesn't slow you down once — it slows down every loop the agent runs, forever. Optimize the inner loop ruthlessly:

| What | Target |
|---|---|
| Unit tests | < 10s |
| Lint | < 5s |
| Single test file | < 3s |
| Dev server start | < 5s |
| Full CI (push → green) | < 5 min |

See [developer-experience/](developer-experience/README.md).

### 2. 🔓 The more the agent has access to, the better
Give the agent the keys. An agent that can read prod (read-only), trigger a build, query the DB, open a browser, and check error logs solves problems end-to-end. One that has to ask you to do each step is a glorified autocomplete.

- Wrap every external system in a `scripts/` command the agent can run. [→ DX scripts](developer-experience/dx-scripts.md)
- Connect MCP servers for issues, browsers, errors, DB access. [→ tools & MCP](ai-agents/tools-and-mcp.md)
- Build a [CodeGraph](developer-experience/codegraph.md) index so it knows the codebase structurally.
- Run with broad permissions + quality-gate hooks instead of approval-prompting every action. [→ hooks & permissions](writing-for-agents/hooks-and-permissions.md)

Access is gated by *safety in the tools* (read-only DB roles, audited gateways, reviewed PRs), not by withholding capability.

### 3. 🐧 Every dev on a Linux VPS with Claude Code
No more "works on my machine." Each developer gets a Linux VPS (or dedicated box) with Claude Code installed and the workspace cloned. **One OS, one setup, one set of commands.**

- Kills the Linux/Windows/Mac fragmentation tax — scripts, paths, and tooling behave identically for everyone.
- Agents run in the same environment they'll deploy to.
- Onboarding is one command. [→ dev VPS standard](developer-experience/dev-vps.md)

### 4. 📦 Monorepo so each part uses the best language for the job
A monorepo lets one product mix runtimes without friction: SolidJS + Bun for the web, Rust for the hot-path API, Swift/Kotlin for native mobile, a C++ core where it matters — all sharing one `CLAUDE.md`, one CI, one set of conventions. Pick the right tool per component; the agent sees the whole picture. [→ monorepo](architecture/monorepo.md)

### 5. 📝 Write everything down — for the next session, not for posterity
Agents are stateless across sessions except for what you write down. A `CLAUDE.md` that documents the exact commands and conventions is worth more than any amount of clever prompting. Compress it (it's read every turn), date load-bearing claims, and delete what rots. [→ writing for agents](writing-for-agents/README.md)

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

---

Next: [✍️ Writing for agents](writing-for-agents/README.md) — the highest-leverage section.
