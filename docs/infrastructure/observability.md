# 🚨 Observability — Sentry as an Agent Work-Queue

Error monitoring isn't just a dashboard humans glance at. In an AI-first org it's a **work queue**: production errors flow in, an agent reads them, fixes the cause, and opens a PR. This is the [fail-fast → fix-fast](../00-philosophy.md) loop in practice.

## Sentry everywhere
Wire **Sentry** (or a compatible error monitor) into every app — frontend, backend, workers, mobile. Capture the error, the stack trace, the release, and enough context to reproduce.

```ts
// backend (Bun/Hono) — init before the app boots
import * as Sentry from "@sentry/bun";
Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.GIT_SHA,        // ties errors to a deploy
  tracesSampleRate: 0.1,
});
```
- **Frontend:** `@sentry/solid` (or browser SDK) — capture unhandled errors + source maps.
- **Mobile:** init in the app entrypoint before the first view mounts ([mobile](../stack/mobile.md)).
- **Tag the release with the git SHA** so an error points straight at the commit that introduced it.

## Init early, fail loud
Start error reporting before anything that can throw, and let errors surface — don't swallow them. This pairs with [low undefined behavior](../00-philosophy.md): typed errors + early capture mean every failure has an obvious where and why.

## Close the loop with an agent
With a **Sentry MCP server** ([tools & MCP](../ai-agents/tools-and-mcp.md)) an agent can:
1. List the top unresolved production errors.
2. Read the stack trace + breadcrumbs.
3. Locate the code (via [CodeGraph](../developer-experience/codegraph.md) / the stack frames).
4. Write a fix **plus a reproducing test**.
5. Open a PR — reviewed by an [automated AI reviewer](../developer-experience/linting-ci.md) and a human.

Run this on a schedule or trigger it on new high-severity issues. The error monitor becomes the front of the autonomous [Planner→Worker→Reviewer](../ai-agents/orchestration.md) loop.

## Beyond errors
- **Structured logs** (JSON) shipped to a sink; trace spans with a `request_id` per call.
- **Metrics** (Prometheus/Grafana) for the cluster and per-app SLOs.
- **Alerts** that escalate to a human (or the deploy agent) on SLO breach — quiet otherwise.

Keep the signal-to-noise high: a flood of warnings trains everyone (and every agent) to ignore the channel.
