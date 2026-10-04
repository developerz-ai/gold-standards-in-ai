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
| **E2E** | real browser → real stack | the handful of journeys that make money or lose data (sign up, pay, core create/edit) | minutes; parallel per worker |

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

## ⏱️ Never `sleep()` in a test — inject the timer

A real wait tests the host's scheduler, not your code. Under load — a coverage run, a CI box with N suites on it — timers coalesce, so a `sleep(3 × interval)` observes 2 beats and the suite goes red for nothing. Then someone "fixes" it with `toBeGreaterThanOrEqual` and the assertion stops meaning anything.

Take the clock as an injectable, next to the other seams, and **drive** it:

```ts
interface IntervalScheduler { every(ms: number, fn: () => void): () => void }

export class Liveness {
  constructor(private deps: { now?: () => number; schedule?: IntervalScheduler } = {}) {}
}

// test: n ticks → n beats. Exact equality. Microseconds, not seconds.
const clock = fakeScheduler();
const l = new Liveness({ now: clock.now, schedule: clock });
clock.tick(3);
expect(beats).toBe(3);
```

Same rule for waiting on **state**: assert the observable value (`client.status === "end"`), never "await this rejects eventually" — a client configured to retry forever never rejects at all, and the test just hangs until the runner's timeout.

If a test needs `sleep`, the code under test is missing a seam. Fix the seam, not the test.

## 🎭 Fake the outside, run the inside for real

| Dependency | In tests | Why |
|---|---|---|
| Your DB, your cache, your queue | **real** (containers / a test DB) | mocks pass while production breaks — the query is the thing under test |
| Third-party HTTP APIs, LLM providers, payment/email/SMS | **fake**, injected at the seam | you can't assert on someone else's uptime, rate limits, or bill |
| Clock, randomness, uuid, filesystem paths | **injected fakes** | determinism |
| Your own other service, over the network | fake the client interface; test the contract separately | keeps the suite fast and hermetic |

Rules that make faking cheap:
- **Constructor/factory DI everywhere.** Tests pass plain-object fakes; nothing patches globals or module registries. **If something is hard to fake, the design is wrong** — move the seam.
- **Fakes, not mocks.** A small real implementation (an in-memory store, a scripted HTTP responder) beats a pile of `expect(spy).toHaveBeenCalledWith(...)` assertions that pin call shapes instead of behavior.
- **One fake per external system, shared across the suite** — in `test/fakes/`, typed by the same interface production implements, so a signature change breaks the fake at compile time.
- **Record real payloads once**, then replay them from a fixture. Fakes drift; a fixture captured from the real API and a periodic contract test against the live sandbox keep them honest.
- **Never let a test hit the network by default.** Fail loudly if it tries — an accidentally-live test is flaky, slow, and occasionally expensive.

## 🔥 Use every core

The suite is the agent's inner loop — it should saturate the machine, not one core of it.

- **Run test files in parallel by default** (Bun does; most runners need a flag). Workers = cores; leave one or two for the dev stack if it's up.
- **Fan out workspaces too**, not just files: run each package's suite concurrently rather than sequentially. On a 14-workspace monorepo that alone is most of the win.
- **Unit tests must be embarrassingly parallel** — no shared files, no fixed ports, no global singletons, no order dependencies. Any test that only passes when it runs first is a bug.
- **Shard across CI runners** for the long tail, and **size the runner to the job** (a formatter on 2 vCPU, the full suite on 8–16) → [smart-builds](../developer-experience/smart-builds.md#4-cache-everything-that-didnt-change).

**Parallel is only safe with per-worker isolation** — otherwise suites truncate under each other and you get wandering failures naming tables the suite never wrote (sometimes even a mid-run auth error):

| Shared thing | Per-worker fix |
|---|---|
| Database | one DB per worker — Postgres clones a migrated template in milliseconds (`CREATE DATABASE … TEMPLATE`) |
| Redis-wire cache | a numbered DB index per worker, flushed in teardown |
| Ports | a base port + worker index, never a constant |
| Build/`target`/browser-profile dirs | a per-worker dir |

Give the harness the worker count and let it provision (`TEST_DB_WORKERS=8`) — an agent should **never** hand-write a connection string. If it's doing that, that's a harness bug.

**The exception: many agents in one checkout.** Then the box is *already* saturated by N agents, and each one fanning out oversubscribes it — the timeouts that follow read as real test failures. Inside a hive: concurrency 1 per agent, explicit file paths, and the coordinator saturates the machine once at the end → [../ai-agents/hive-mind.md](../ai-agents/hive-mind.md).

## 🧭 E2E — committed journeys, not ad-hoc clicking

An agent poking the UI through Playwright MCP *sees* a bug once; a committed spec catches it forever. Every key journey is a Playwright spec in the repo, run by the gate.

| Rule | Do | Never |
|---|---|---|
| Locators | `getByRole("button", { name: "Pay" })`, `getByLabel("Email")` — doubles as an a11y check → [../frontend-craft/accessibility.md](../frontend-craft/accessibility.md) | CSS paths, `nth-child`, text that changes with copy |
| Waiting | wait for the **thing**: `await page.waitForResponse(r => r.url().includes("/api/orders") && r.ok())`, `await expect(locator).toBeVisible()` | `page.waitForTimeout(…)` — same disease as `sleep()` above |
| Failure output | `trace: "retain-on-failure"` + screenshot, saved as a CI artifact the agent opens (`npx playwright show-trace`) | a red line with no artifact |
| Visual check | compare to a committed baseline; **no baseline = INCONCLUSIVE**, never PASS | auto-accepting the first screenshot as truth |
| Flakes | fix the wait or the race; flaky policy → [../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md) | `retries: 2` in CI — it hides the race the user will hit |

```ts
// playwright.config.ts
export default defineConfig({
  retries: 0,
  use: { trace: "retain-on-failure", screenshot: "only-on-failure" },
});
```

**Post-deploy smoke** — the same journeys, read-only, against the live URL after every rollout. Fail on: any `console.error`, any `pageerror`, any 4xx/5xx from your own origin.
```ts
page.on("console", m => m.type() === "error" && errors.push(m.text()));
page.on("pageerror", e => errors.push(e.message));
page.on("response", r => r.url().startsWith(BASE) && r.status() >= 400 && errors.push(`${r.status()} ${r.url()}`));
// … journey …
expect(errors).toEqual([]);
```

**Click-path audit** — for bugs where "the button does nothing". Per touchpoint, trace each store action the handler calls and what it **sets and resets**:
```
selectThread(t) → sets {selectedThread} RESETS {composeMode}
"New email" onClick: setComposeMode(true); selectThread(null)  ← second call undoes the first
```
Hunt: a later call resetting an earlier one · two async results racing on one field · an effect that reverts the state just set · a handler whose name promises an API call it never makes · a branch guarded by state that is always false there. The fix lands with an E2E spec for that journey.

## TDD where it fits
Convert the task to a failing test first:
- "Fix the bug" → reproducing test → make it pass.
- "Add validation" → tests for invalid inputs → make them pass.
- "Refactor X" → tests green before AND after.

**A reproducing test counts only once it ran RED on the unfixed code** — executed, and failing for the bug's reason (not a typo, a missing import, a broken fixture). Written-but-never-run, or first run after the fix, proves nothing: it may pass on the old code too. Paste the red run in the PR.

This is the engine behind [goal-driven execution](../writing-for-agents/behavioral-rules.md) — the agent loops edit→run→check until green.

Reviewing the tests themselves (untested error paths, assertions that only check "doesn't throw", mocks of the thing under test) → test-quality lens in [../writing-for-agents/reviewer-agents.md](../writing-for-agents/reviewer-agents.md#-specialist-lenses-run-them-in-parallel).

## Coverage as a gate (optional)
Where it matters, block merges under a threshold (e.g. line + branch ≥ 90%). Keep it honest — coverage is a floor, not a goal.

## Document the commands in CLAUDE.md
```bash
bun test                       # all unit tests
bun run test:integration       # integration (after bin/dev)
bun test path/to/file.test.ts  # one file
bunx playwright test           # E2E journeys (after bin/dev)
```
If running a single focused test is slow or awkward, fix that — the agent runs it constantly. See [DX](../developer-experience/dx-scripts.md).
