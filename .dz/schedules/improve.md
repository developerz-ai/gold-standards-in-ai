# Brief: monthly improvement

Fired on the first Monday of the month by `schedules: improve` (`weeks_of_month: [1]`).

Once a month, keep what agents read here true. `CLAUDE.md` binds every change, and `non_goals` in the policy file are off limits. A red `main` belongs to the platform's red-main ask; broken links and an unindexed doc belong to the gate (`bin/check`).

## Signals

| Signal | Command | Worth acting on |
|---|---|---|
| dated claims | `grep -rn 'As of 20' README.md docs` | a claim older than three months that a source now contradicts |
| external links | `git ls-files '*.md' \| xargs grep -ohE 'https://[^) >]+' \| sort -u`, then `curl -sIL --max-time 10` each | a 404 or a domain gone; a transient failure is not a finding |
| model ids and versions | the same grep for `claude-` | a model id or tool version no longer current |
| stuck Dependabot PR | `gh pr list --state open --author app/dependabot --json number,title,createdAt,url` | an open PR older than 7 days: fix or close it with the reason |
| High advisory for the pinned Sentry MCP server | `gh api 'advisories?ecosystem=npm&affects=@sentry/mcp-server&per_page=100'` | a `high` advisory whose vulnerable range includes the `.mcp.json` pin: bump the pin |

## Pick at most three, in this order

1. A broken link or a claim a source now contradicts: fix it, and cite the source in the PR.
2. A stale doc nothing can correct: delete it and its index lines (`CLAUDE.md`: delete rather than let it rot).
3. Compression: a doc that drifted from the writing rules in `CLAUDE.md`.

## Ship

One small PR per improvement, its evidence in the body, `bash bin/check` green first. Follow the writing rules in `CLAUDE.md`.
Check no open PR or issue already covers it. If nothing is worth changing, do nothing: no empty PRs, no churn.

## Rows

- `dated-claims` — Every load-bearing claim carries a date; use the existing `grep -rn 'As of 20' README.md docs` command in the Signal table above.
- `external-links` — Every external link resolves; use the existing external-link grep and `curl -sIL --max-time 10` in the Signal table above.
- `stuck-dependabot` — No open Dependabot PR older than 7 days; use the Dependabot command in the Signal table above.
- `sentry-mcp-advisory` — No High advisory affects the `.mcp.json` Sentry MCP pin; use the advisory command in the Signal table above.
- `model-ids` — Every model id and tool version is current; use the existing `claude-` grep in the Signal table above.
