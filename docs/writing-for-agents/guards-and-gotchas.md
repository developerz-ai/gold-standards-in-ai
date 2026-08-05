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
- **Allowlist, don't weaken.** Pre-existing violations go in an explicit `<rule>-allowlist.ts` that shrinks over time — never loosen the rule to fit legacy code.
- **Born from a real incident.** Guard the defect that happened, not the one you imagine.

The payoff is compounding: an agent that writes the forbidden shape learns *at lint time, in its own loop*, with no human in the room.

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

**Related:** [hooks-and-permissions.md](hooks-and-permissions.md) — the iteration rule · [../developer-experience/linting-ci.md](../developer-experience/linting-ci.md) — wiring guards into the gate · [../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md) — local gate ≡ CI
