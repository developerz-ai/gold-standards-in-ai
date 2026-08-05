# ⏱️ The Inner Loop — scoped checks, not the full gate

The agent runs checks **dozens of times per task**. If the only gate you expose is the whole-repo one, every one of those runs pays for 14 workspaces it didn't touch — and a multi-minute foreground command *looks hung*, so the agent starts guessing instead of verifying.

**Ship three tiers, and say in `CLAUDE.md` which is which.**

| Tier | Command | Scope | Budget | When |
|---|---|---|---|---|
| **Tightest** | `bun run check:files <paths>` | only the files you edited: formatter + every path-scoped guard | < 3s | after each edit |
| **Scoped** | `typecheck:changed` · `test:changed` | only the workspaces your diff touches | < 30s | after a slice compiles |
| **Full gate** | `bin/check` / `bun run verify` | every workspace + every compiled language — the CI mirror | minutes | **once**, before push, in the **background** |

State the anti-rule explicitly, because the agent's instinct is to reach for the biggest hammer:

```markdown
Prefer the scoped, separate commands. Do NOT reflexively run `bun run verify` —
it typechecks + tests all workspaces AND compiles the Rust services. Run it once,
right before pushing, in the background.
```

## How "changed" is computed

Diff the working tree against the merge base and map changed files → workspaces, with one shared helper — the **local twin of CI's `detect` job**:

```ts
// scripts/dev/affected.ts — used by typecheck:changed, test:changed, and CI
const files = await git(`diff --name-only origin/main...`);
const workspaces = files.map(toWorkspace).filter(Boolean);
```

One implementation, two consumers. If local scoping and CI scoping drift, "green locally" stops meaning anything → [ai-first-cicd.md](ai-first-cicd.md#the-parity-test).

A TS-only change must never trigger a Rust compile, and vice versa. That single rule is most of the speedup on a polyglot [monorepo](../architecture/monorepo.md).

## Name the blind spots — scoping hides real breakage

`:changed` is **scoped and unit-only**. Say so where the agent reads it:

- A break in an unaffected workspace won't show.
- A break only an **integration** test covers won't show — run those explicitly for what you touched.
- A break in the compiled languages won't show unless their files changed.

That's an acceptable trade *because* the full gate runs before push. Undocumented, it's a trap.

## 🚨 Scoped ≠ safe when agents share the checkout

`:changed` diffs the **working tree**, and in a [hive](../ai-agents/hive-mind.md) that tree holds *everyone's* uncommitted work. Each "scoped" run expands to nearly the whole repo, N of them run at once, and they contend on one test DB. (2026-07, real sweep: 2-minute checks became 30-minute ones.)

Inside a hive, the legal commands are **explicit paths only**:

```bash
bun run check:files apps/api/src/routes/foo.ts apps/api/test/unit/foo.test.ts
bun test apps/api/test/unit/foo.test.ts
```

Concurrency 1 per agent; the repo-wide gate belongs to the coordinator, once, at the end.

## Make a single test cheap

The agent runs one test file more often than any other command. If that path is awkward, everything downstream is slower.

- **One file, from a cold shell, no ceremony**: `bin/dev test path/to/x.test.ts` loads the dev env for you.
- **Run workspace commands from the repo root** so root-level `.env.test` loads — otherwise the agent debugs a missing env var instead of its change.
- **Self-provisioning stores.** The test harness creates/migrates its own database; an agent should never hand-write a `DATABASE_URL`. If it's doing that, that's a harness bug.
- **Isolate per worker** (`TEST_DB_WORKERS`, numbered cache DBs) or parallel suites truncate under each other and surface as failures naming tables the suite never wrote → [../architecture/testing.md](../architecture/testing.md).
- **Frontend needs its own conditions flag?** Put it in the workspace script, not in the agent's head — a raw runner invocation showing ~90 phantom failures burns a whole turn.

## Fail one, fix one

Run the tiers **in order, stop at the first failure, fix, re-run only that command.** Chaining everything with `&&` and re-running the chain wastes the fast tiers on a known-failing state.

And never trust a piped exit code — `cmd | tail; echo $?` reports `tail`'s status. Run the tool directly, or check `${PIPESTATUS[0]}`.

## Put the loop in a slash command

Wrap the tiers in `/check` so it's one keystroke and the *report format* is fixed:

```markdown
---
description: Run the fast scoped checks. Report failures concisely and fix them.
---
1. `bun run typecheck:changed`
2. `bun run test:changed`
3. `bun run lint`
Stop at the first failure, fix it, re-run only that command.
Report: one line per command — `✓ typecheck:changed` or `✗ typecheck:changed — <error summary>`.
Fix rules: type errors → fix the source, never cast to `any`. Lint → `lint:fix` first, then fix what remains by hand.
```

---

**Related:** [dx-scripts.md](dx-scripts.md) — the `bin/` trio · [ai-first-cicd.md](ai-first-cicd.md) — the same commands in CI · [linting-ci.md](linting-ci.md) — Biome + guards
