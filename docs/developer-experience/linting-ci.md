# ✨ Linting & CI

Fast, automated, non-negotiable. The agent should never make a formatting decision, and CI should never be the bottleneck.

## Biome — one Rust binary
Biome replaces ESLint + Prettier for all TS/JS. It's ~10–100× faster, so the lint gate never slows the [inner loop](../00-philosophy.md).

```json
{
  "$schema": "https://biomejs.dev/schemas/2.4.16/schema.json",
  "vcs": { "enabled": true, "clientKind": "git", "useIgnoreFile": true },
  "formatter": { "enabled": true, "indentStyle": "space", "indentWidth": 2, "lineWidth": 100 },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "suspicious": { "noExplicitAny": "error" },
      "style": { "useImportType": "error", "noNonNullAssertion": "warn" },
      "correctness": { "noUnusedVariables": "error", "noUnusedImports": "error" }
    }
  },
  "javascript": { "formatter": { "quoteStyle": "single", "semicolons": "always", "trailingCommas": "all" } },
  "assist": { "actions": { "source": { "organizeImports": "on" } } },
  "files": { "includes": ["**", "!**/node_modules", "!**/dist", "!**/migrations"] }
}
```
Commands: `biome check .` (lint) · `biome check --write .` (fix). Wire into `bun run lint` / `lint:fix`.

Other languages: **Rubocop** (Ruby), **`cargo fmt` + `clippy -D warnings`** (Rust), `gofmt` (Go). One `bin/lint` runs them all.

## Blacksmith CI runners
Run GitHub Actions on **Blacksmith** runners (`blacksmith-2vcpu-ubuntu-2404`) — faster, consistent, no queue waits.

```yaml
name: CI
on:
  push: { branches: [main] }
  pull_request:
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
jobs:
  check:
    runs-on: blacksmith-2vcpu-ubuntu-2404
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v7
      - uses: useblacksmith/setup-bun@v2
        with: { bun-version: latest }
      - run: bun install --frozen-lockfile
      - run: bun run lint
      - run: bun run typecheck
      - run: bun test
```

## Build & deploy pipeline
On merge to `main`: build a Docker image, push to a registry — **GHCR** (`ghcr.io`, public) or **DOCR** (DigitalOcean Container Registry, private) — tagged `latest` + `<sha>`. Deployment is pulled in-cluster by ArgoCD Image Updater — **CI holds zero cluster credentials**, the handoff is image-only. Registries + hosting: [../infrastructure/servers-and-dns.md](../infrastructure/servers-and-dns.md) · pipeline: [../infrastructure/kubernetes-gitops.md](../infrastructure/kubernetes-gitops.md).

```yaml
# deploy.yml (matrix per app)
strategy:
  matrix:
    include:
      - { image: product-api, dockerfile: docker/api.Dockerfile }
      - { image: product-web, dockerfile: docker/web.Dockerfile }
steps:
  - uses: docker/build-push-action@v5
    with:
      push: ${{ github.ref == 'refs/heads/main' }}
      tags: |
        ${{ env.REGISTRY }}/${{ env.PREFIX }}/${{ matrix.image }}:latest
        ${{ env.REGISTRY }}/${{ env.PREFIX }}/${{ matrix.image }}:sha-${{ github.sha }}
```

## CI/CD is the bottleneck — cache everything
The agent waits on CI before it can merge or deploy, so CI latency taxes every loop. Attack it:

- **Cache dependencies.** Restore Bun's install cache keyed on the lockfile; `bun install --frozen-lockfile` then becomes near-instant.
- **Cache the build.** Docker **layer caching** (BuildKit / `cache-from` + `cache-to` to a registry or Blacksmith's sticky disk) — never rebuild a layer that didn't change.
- **Cache test/compile artifacts** — Rust `target/`, TS build info, incremental builders (esbuild/swc over webpack).
- **Pre-built base images** — bake system deps into a base image; don't `apt-get install` on every run.
- **Blacksmith** runners give fast SSDs + sticky caches out of the box → fewer cold starts.

```yaml
- uses: useblacksmith/setup-bun@v2          # cached bun
  with: { bun-version: latest }
- run: bun install --frozen-lockfile        # restored from cache
- uses: docker/build-push-action@v5
  with:
    cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ matrix.image }}:buildcache
    cache-to:   type=registry,ref=${{ env.REGISTRY }}/${{ matrix.image }}:buildcache,mode=max
```

## Detect changes — only do the work that's needed
Don't test or build the whole monorepo on every PR. Detect which packages/apps changed (path filters, `turbo --filter`, or `git diff` against the merge base) and run only the affected pipelines. The full treatment — one shared detector, build-vs-roll content hashing, per-unit image repos, cache-key design — is [smart-builds.md](smart-builds.md).

```yaml
# build only images whose context changed
- id: changes
  uses: dorny/paths-filter@v3
  with:
    filters: |
      api: ['apps/api/**', 'packages/**']
      web: ['apps/web/**', 'packages/ui/**']
# downstream jobs gate on steps.changes.outputs.<name> == 'true'
```
Combine with the deploy matrix so an unchanged app is never rebuilt — the single biggest CI win on a large monorepo.

## Parallel tests — super fast
- **Bun** runs test files in parallel by default; split big suites across CI workers with a matrix shard.
- **Isolate stateful stores per worker** (own DB index, `FLUSHDB`/truncate in teardown) so parallelism is safe → [../architecture/testing.md](../architecture/testing.md).
- Keep the unit suite under ~10s and the full gate under ~5 min — a slow suite slows every agent loop.

## The image builder
On merge to `main`, a matrix job builds one image per app (with the caching above), tags `latest` + `<sha>`, and pushes to the registry. Multi-arch (amd64 + arm64) via Buildx when targets differ. The cluster pulls the new digest itself — CI never holds cluster credentials → [../infrastructure/kubernetes-gitops.md](../infrastructure/kubernetes-gitops.md).

## Speed targets
| Stage | Target |
|---|---|
| Lint | < 5s |
| Typecheck | < 15s |
| Unit suite | < 10s |
| Full CI (push → green) | < 5 min |
| Docker build | < 3 min (layer caching) |

## Automated AI code review
Put an **AI reviewer on every PR** — CodeRabbit works well. It catches bugs, suggests improvements, writes the high-level summary, and enforces repo-specific rules *before* a human looks. With an [autonomous agent](../ai-agents/orchestration.md) opening PRs, an automated reviewer is the second pair of eyes that keeps the loop honest at machine speed.

Configure it with path-scoped instructions so it knows your conventions:
```yaml
reviews:
  profile: chill
  path_instructions:
    - path: "packages/domain/**"
      instructions: "Pure types only. Flag any I/O (fs, network, db, env)."
```
The human still merges. Pin third-party actions to a commit SHA, and run a secret-scan job on PRs.
