# 🛡️ Guards & Gotchas — don't tell the next agent to be careful

**Make the machine careful.** A rule in prose is advice; a script that fails CI is a fact. Every defect an agent can repeat should end its life as something mechanical — because the next agent has no memory of the incident, only what's in the repo.

> A comment saying "be careful here" is a bug report against your tooling.

## The ladder — pick the cheapest rung that actually binds

| Rung | Home | Use when |
|---|---|---|
| 1. **Type / schema** | the code | the illegal state can be made unrepresentable (`type X = undefined`, a Zod refinement, a branded id) |
| 2. **Assertion at the seam** | the one function the bad value flows through | there is a single entry point — throw there, naming the value and the fix |
| 3. **Contract test** | `test/` | an invariant spans files (route ↔ nginx, queue name ↔ worker, schema ↔ fixture) |
| 4. **Custom lint guard** | `scripts/lint/<rule>.ts` + CI | a *shape* of code is forbidden anywhere in the repo |
| 5. **Doctor check** | `bin/dev doctor` | the environment can be wrong (port drift, missing env key, pending migration) |
| 6. **Preventive rule** | `CLAUDE.md` | no mechanism exists yet and getting it wrong is expensive |
| 7. **Gotcha entry** | `docs/gotchas.md` | it's a *symptom lookup* — "this error message means that cause" |

Climb as high as you can afford **in the same PR as the fix**. A rule that only lives on rungs 6–7 will be violated again; a rule on rungs 1–4 cannot be.

## Custom lint guards — institutional memory that executes

Biome catches style. **Guards catch your repo's specific ways of dying.** One file per rule, one rule per file, each with its own test:

```
scripts/lint/
├── no-bare-error.ts            # every throw uses a domain error class
├── no-bare-error.test.ts       # the guard's own test: flags bad, passes good
├── max-file-loc.ts             # files ≤500 LOC
├── no-unbounded-sweep-read.ts  # a background read without a LIMIT/window
├── no-seed-in-migration.ts     # migrations are DDL; business rows go to the seed layer
├── api-service-wiring.ts       # every service the routes need is actually injected
└── migration-compat.ts         # dialect features prod's engine doesn't have
```

Rules for a guard:
- **It fails CI**, or it isn't a guard. Wire every one into `bun run lint`.
- **Test the guard itself** — a guard that silently matches nothing is worse than none. `<rule>.test.ts` asserts it flags a bad sample *and* passes a good one.
- **Error text names the value, the seam, and the fix**: `bullmq id "sync:org" contains ':' — use '-' (queues.ts:187). ':' is BullMQ's key separator; the sweep would enqueue 0 jobs.`
- **Allowlist, don't weaken.** Pre-existing violations go in an explicit `<rule>-allowlist.ts` that shrinks over time — never loosen the rule to fit legacy code. Enforced by the [ratchet](#-dont-weaken-the-gate--a-ratchet-not-a-rule).
- **Born from a real incident.** Guard the defect that happened, not the one you imagine.

The payoff is compounding: an agent that writes the forbidden shape learns *at lint time, in its own loop*, with no human in the room.

## 🔩 Don't weaken the gate — a ratchet, not a rule

An agent stuck on a red check has a cheaper move than fixing the code: loosen the check. "Allowlist, don't weaken" is prose until a guard enforces it. `scripts/lint/gate-ratchet.ts` diffs HEAD against the merge base and **fails when the gate got weaker**:

| Weakening | Detected by |
|---|---|
| A rule in `biome.json` removed or lowered (`error` → `warn`/`off`) | parse both versions, compare per-rule severity |
| `tsconfig*.json` strictness off (`strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, …) | parse both, compare flags |
| An `*-allowlist.ts` grew | entry count, base vs HEAD |
| More suppressions | count of `biome-ignore`, `@ts-expect-error`, `@ts-ignore`, `#[allow(`, `.skip(` / `.only(` — base vs HEAD |
| `trustedDependencies` grew | `package.json` array length |

Escape hatch is **evidence, not permission**: the guard passes if the PR body carries `Gate-change: <why the rule is wrong here>`. CI reads it from `github.event.pull_request.body`; the reviewer sees the line in the diff summary. Shrinking any count always passes — the ratchet only turns one way.

## 🔇 Silent-failure guards — errors must surface

The bug an agent writes most and a test catches least: the error that never reaches anyone. Mechanize the shape; leave the judgment ("is this fallback honest?") to the [reviewer agent's silent-failure lens](reviewer-agents.md#-silent-failure-lens-our-top-hunt-target).

| Shape | Guard |
|---|---|
| Empty `catch {}` | Biome `suspicious.noEmptyBlockStatements: "error"` — a comment silences it, so the comment must say *why continuing is correct* |
| `catch` whose body is only `return []` / `null` / `{}` / `false` | custom `scripts/lint/no-swallowed-error.ts` — no rethrow, no log, no domain error → fail |
| Un-awaited promise | Biome `nursery.noFloatingPromises: "error"` (types domain, nursery as of 2026-10) |
| Rust `let _ = fallible()` | `clippy::let_underscore_must_use` + `#[must_use]` on your `Result`-returning fns |

A fallback that hides a failure is a lie the next agent debugs for an hour. Return the error, or log it with the value that caused it.

## 🧪 Repo-poisoning guard — the agent-read surface is code

Every file an agent reads is an instruction channel. A PR (or a dependency's vendored skill) can hide instructions a human reviewer never sees. Why → [../ai-agents/untrusted-input.md](../ai-agents/untrusted-input.md). `scripts/lint/agent-surface.ts` scans `CLAUDE.md`, `**/CLAUDE.md`, `AGENTS.md`, `.claude/**`, `skills/**`, `.mcp.json` and fails on:

| Fail on | Why |
|---|---|
| Zero-width / bidi control chars (`U+200B–U+200D`, `U+2060`, `U+FEFF`, `U+202A–U+202E`, `U+2066–U+2069`) | invisible to the reviewer, read by the model |
| HTML comments (`<!-- … -->`) not on the allowlist | renders as nothing on GitHub, reaches the model verbatim |
| `ANTHROPIC_BASE_URL` in any committed settings `env` | reroutes API traffic — and the key — to someone else's endpoint |
| `enableAllProjectMcpServers` in committed settings | a PR that adds a server to `.mcp.json` would auto-start it on every box; approve per box in the gitignored `.claude/settings.local.json` |

```bash
rg -nP '[\x{200B}-\x{200D}\x{2060}\x{FEFF}\x{202A}-\x{202E}\x{2066}-\x{2069}]' CLAUDE.md AGENTS.md .claude skills .mcp.json
rg -n '<!--|ANTHROPIC_BASE_URL|enableAllProjectMcpServers' CLAUDE.md AGENTS.md .claude skills .mcp.json
```

## `bin/dev doctor` — the environment guard

Half of "the agent is broken" is "the box is wrong." One command, one truthful row per thing that can drift:

```
✔ postgres      reachable  :5433
✖ dragonfly     port drift — compose published :6379, .env says :6382
✔ env           all required keys present
✖ migrations    3 pending — run bun run db:migrate
```

Rules: **one row per store**, real probes not `docker ps`, and it must be *honest* — "reachable" only if a query returned. Add a check the first time a wrong environment costs someone an hour.

## Where knowledge lands: `CLAUDE.md` vs `docs/gotchas.md`

Two files, one split — get it wrong and `CLAUDE.md` bloats into a runbook.

| | `CLAUDE.md` — **preventive rules** | `docs/gotchas.md` — **diagnostic runbook** |
|---|---|---|
| Read | every turn | when a symptom appears |
| Contains | rules that bite **before** any symptom | symptom → cause → fix |
| Test | "would an agent write the bug without this?" | "would an agent search this after seeing the error?" |
| Example | "Never put `:` in a queue id — the sweep silently enqueues 0 jobs." | "`FETCH` returns empty on this IMAP server → known upstream bug, use …" |

Write preventive rules as **mechanism, not manners**: state what breaks, when it surfaces, and what it costs *then*. `"Never hand-edit the migration journal timestamp — a future stamp makes the runner skip every later migration, silently, in prod."` beats "be careful with the journal."

## The threshold — when to automate

**Hit twice, or cost real time once → automate.** Once, cheaply, with no named mechanism of failure → don't; you'd be guarding a coincidence.

Tells that you're overdue:
- The same three commands, again.
- Debug prints added, then deleted.
- Setup knowledge living only in one person's head.
- A command that answers with silence instead of an error.
- Checking by eye what a machine could assert.

## Say it where the agent is standing

Placement beats phrasing. A rule about migrations belongs in `packages/db/CLAUDE.md`; a rule about fan-out belongs inside the `/feature` command file; a rule about a queue id belongs in the error the enqueue seam throws. The root `CLAUDE.md` carries only what applies everywhere → [claude-md.md](claude-md.md#keep-it-small--index-dont-inline).

## Proceeding anyway? Leave evidence.

When a known-wrong shortcut ships on purpose, the debt must be impossible to lose: a **pinning test** that asserts the known-bad behavior (so changing it is a deliberate act) or a **`docs/gotchas.md` line**. Never a bare `TODO` — nobody greps those.

---

**Related:** [hooks-and-permissions.md](hooks-and-permissions.md) — the iteration rule · [reviewer-agents.md](reviewer-agents.md) — the judgment half of silent failures · [../ai-agents/untrusted-input.md](../ai-agents/untrusted-input.md) — why the agent-read surface is attack surface · [../developer-experience/linting-ci.md](../developer-experience/linting-ci.md) — wiring guards into the gate · [../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md) — local gate ≡ CI
