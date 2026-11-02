# Brief: monthly improvement

Fired every Monday by `schedules: improve`. **Act only on the first Monday of the month:**
`[ "$(date -u +%-d)" -le 7 ]`. On any other Monday stop here, file nothing, open nothing.

Once a month, keep what agents read here true. `CLAUDE.md` binds every change, and `non_goals` in the policy file are off limits. A red `main`,
a missing publish and a stuck Dependabot PR belong to `health`.

## Signals

| Signal | Command | Worth acting on |
|---|---|---|
| dated claims | `grep -rn 'As of 20' README.md docs` | a claim older than three months that a source now contradicts |
| external links | `git ls-files '*.md' \| xargs grep -ohE 'https://[^) >]+' \| sort -u`, then `curl -sIL --max-time 10` each | a 404 or a domain gone; a transient failure is not a finding |
| model ids and versions | the same grep for `claude-` | a model id or tool version no longer current |

## Pick at most three, in this order

1. A broken link or a claim a source now contradicts: fix it, and cite the source in the PR.
2. A stale doc nothing can correct: delete it and its index lines (`CLAUDE.md`: delete rather than let it rot).
3. Compression: a doc that drifted from the writing rules in `CLAUDE.md`.

## Ship

One small PR per improvement, its evidence in the body, `the health loop` green first. Follow the writing rules in `CLAUDE.md`.
Check no open PR or issue already covers it. If nothing is worth changing, do nothing: no empty PRs, no churn.
