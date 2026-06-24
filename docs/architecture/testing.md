# 🧪 Testing

Tests are non-negotiable in AI-first development — they're how the agent **verifies its own work**. No tests = the agent is flying blind on every refactor.

Three jobs tests do for an agent:
- **Confidence** — catch regressions from aggressive refactors.
- **Specification** — document expected behavior the agent reads.
- **Verification** — a command that proves "done" after every change.

## The pyramid
| Kind | Hits | Use for | Speed target |
|---|---|---|---|
| **Unit** | pure logic, no I/O | validators, formatters, domain logic | < 10s suite, < 3s file |
| **Integration** | real services (DB, cache) | queries, jobs, API endpoints | minutes OK |
| **Property** | generated inputs | invariants ("most restrictive grant wins") | as needed |

## Unit — Bun's native runner
```ts
import { describe, expect, it } from "bun:test";
import { validateHandle } from "./handle";

describe("validateHandle", () => {
  it("accepts lowercase with hyphens", () => {
    expect(validateHandle("my-handle")).toBe(true);
  });
});
```
Solid components test with `@solidjs/testing-library`:
```ts
import { render } from "@solidjs/testing-library";
const { getByText, unmount } = render(() => <Onboarding />);
expect(getByText("Get started")).toBeTruthy();
unmount();
```

## Integration — real services via TestContainers
Don't mock the database — mocks pass while production breaks.
```ts
import { PostgreSqlContainer } from "@testcontainers/postgresql";
import { RedisContainer } from "@testcontainers/redis";

const pg = await new PostgreSqlContainer().start();
const redis = await new RedisContainer().start();
// run migrations against pg.getConnectionUri(), then assert
await pg.stop(); await redis.stop();
```
For Rust: integration tests boot the service against a real Postgres CI service container and issue real requests.

## Parallel + isolated
- **Bun** runs test files in parallel out of the box.
- **Per-worker isolation** for stateful stores: give each parallel worker its own DB index (e.g. Redis DB 1–14), `FLUSHDB` in teardown so every test starts clean.
- Split big suites across CI workers to keep the gate under ~5 min.

## TDD where it fits
Convert the task to a failing test first:
- "Fix the bug" → reproducing test → make it pass.
- "Add validation" → tests for invalid inputs → make them pass.
- "Refactor X" → tests green before AND after.

This is the engine behind [goal-driven execution](../writing-for-agents/behavioral-rules.md) — the agent loops edit→run→check until green.

## Coverage as a gate (optional)
Where it matters, block merges under a threshold (e.g. line + branch ≥ 90%). Keep it honest — coverage is a floor, not a goal.

## Document the commands in CLAUDE.md
```bash
bun test                       # all unit tests
bun run test:integration       # integration (after bin/dev)
bun test path/to/file.test.ts  # one file
```
If running a single focused test is slow or awkward, fix that — the agent runs it constantly. See [DX](../developer-experience/dx-scripts.md).
