# 🏃 DX Scripts

Every project exposes the same handful of commands. The agent (and any human) learns one repo and knows them all. Great DX = great agents.

## The core trio
| Command | Does | Run |
|---|---|---|
| `bin/setup` | prerequisites → `bun install` → `.env` → boot Docker services → migrate | once, after clone |
| `bin/dev` | bring up the full local stack with hot reload | every session |
| `bin/check` | lint + typecheck + unit tests (the CI gate, locally) | before commit |

Plus: `bin/fmt` (auto-format), `bun run db:migrate`, `bun run db:seed`, `bun run test:integration`.

## `bin/setup` — fresh clone to running
```bash
#!/usr/bin/env bash
set -euo pipefail
ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"; cd "$ROOT"

ok(){ echo -e "\033[0;32m✔\033[0m  $*"; }

echo "▸ Checking prerequisites"
command -v bun    >/dev/null || { echo "bun not found"; exit 1; }
command -v docker >/dev/null || { echo "docker not found"; exit 1; }
ok "bun $(bun --version), docker present"

echo "▸ Installing deps"; bun install; ok "deps installed"

echo "▸ Env"
[ -f .env.development.local ] || cat > .env.development.local <<'EOF'
# Per-box overrides + secrets — GITIGNORED, wins over .env.development
EOF

echo "▸ Starting data stores"
docker compose -f docker/docker-compose.yml up -d
ok "Postgres/Yugabyte + Dragonfly up"

echo "▸ Waiting for DB, then migrating"
until docker compose -f docker/docker-compose.yml exec -T db pg_isready >/dev/null 2>&1; do sleep 2; done
bun --filter @product/db db:migrate
ok "Setup complete. Next: bin/dev"
```

## `bin/dev` — boot what you're working on
```bash
#!/usr/bin/env bash
set -euo pipefail; cd "$(dirname "$0")/.."
target="${1:-all}"
case "$target" in
  web)     exec bun run --filter @product/web dev ;;
  api)     exec bun run --filter @product/api dev ;;
  all)     trap 'kill $(jobs -p) 2>/dev/null' EXIT
           bun run --filter @product/api dev &
           bun run --filter @product/web dev &
           wait ;;
  *) echo "usage: bin/dev [web|api|all]"; exit 2 ;;
esac
```

## `bin/check` — the gate
```bash
#!/usr/bin/env bash
set -euo pipefail; cd "$(dirname "$0")/.."
bun run lint
bun run typecheck
bun run test
```
Wire the same commands into [CI](linting-ci.md) and a [pre-commit hook](../writing-for-agents/hooks-and-permissions.md) so local, hook, and CI agree.

## Alternative runner: justfile
For smaller repos a `justfile` works well:
```just
default: @just --list
setup:     bun install
dev:       bun run dev
verify:    bun run lint && bun run typecheck && bun run test
fix:       bun run lint:fix
```

## Committed defaults = zero-config boot
Commit non-secret defaults so `bun run dev` works with no manual setup:
- `.env.development` — non-secret local defaults (committed)
- `.env.development.local` — per-box secrets/overrides (gitignored, wins)
- `biome.json`, `drizzle.config.ts`, `tailwind.config.js`, `docker/docker-compose.yml`

## Scripts are Bun TypeScript, not bash
`bin/*` entrypoints are tiny shell shims, but the real automation lives in **Bun TS** under `scripts/`. TypeScript means types, `bun test`, and no fragile string-munging — and it's the same language as the rest of the stack.

```
scripts/
├── <resource>/<verb>.ts     # one verb per file: servers/list.ts, dns/upsert.ts, mail/create.ts
├── lib/                      # reusable modules — the shared brain (≤200 LOC, pure where possible)
│   ├── args.ts              # flag parsing
│   ├── cmd.ts               # run / runOrThrow subprocess helpers
│   ├── log.ts               # leveled logging
│   ├── <provider>/client.ts # ALL HTTP for a provider goes through one client
│   └── ssh/{exec,scp}.ts    # ALL SSH goes through here (ProxyJump-aware)
└── help.ts                  # prints the full catalog
```

Run them directly: `bun scripts/dns/upsert.ts example.com A @ 1.2.3.4`.

### Rules for `scripts/`
- **One verb per file.** `scripts/<resource>/<verb>.ts`.
- **`scripts/lib/**/*.ts` is reusable code** — SRP modules, ≤200 LOC, pure where possible. Don't duplicate logic across scripts; lift it to `lib/`.
- **No inline HTTP** — every external API call goes through `lib/<provider>/client.ts`.
- **No inline SSH / no inline bash / no hand-parsed YAML** — wrap each in a `lib/` helper with one well-tested implementation.
- **Typecheck the scripts too:** `tsc -p scripts/tsconfig.json` in `bin/check`.
- **`lib/` is unit-tested** — pure helpers are the easiest, highest-value tests in the repo → [../architecture/testing.md](../architecture/testing.md).

## A `./tmp/` scratch dir
Give the agent a gitignored `./tmp/` for intermediate work — generated keys, scratch scripts, downloaded artifacts, command output it wants to re-read, a place to stage files before moving them. It keeps junk out of the source tree and out of commits.
```gitignore
# .gitignore
/tmp/
```
- One per repo (and the agent's own session scratch dir for truly throwaway files).
- Anything that should persist graduates out of `tmp/` into a real path; everything else is safe to wipe.
- Never put secrets meant for commit here; `tmp/` is disposable by definition.

## Rules
- **Wrap every external system** (hosting, CI, DNS, DB, monitoring) in a script the agent can run — this is how it works end-to-end ([philosophy #2](../00-philosophy.md)).
- **Read-only where it counts** — DB scripts use a read-only role.
- **Add `scripts/*` and `bin/*` to the [allow list](../writing-for-agents/hooks-and-permissions.md)** so the agent runs them without prompts.
