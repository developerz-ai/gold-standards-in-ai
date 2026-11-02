---
description: Monthly upkeep of dated claims, external links and model ids. Use for stale doc, dated claim, model id, monthly improve.
---

> House skill — established by dz-runner setup mode from this repo's own evidence, then owned by its maintainers. Correct it in place; setup never overwrites a skill that exists.

1. Act only on the first Monday of the month: `[ "$(date -u +%-d)" -le 7 ]`. fail -> stop, file nothing, open nothing.
2. Dated claims: `grep -rn 'As of 20' README.md docs` -> a claim older than three months that a source now contradicts is worth acting on.
3. External links: `git ls-files '*.md' | xargs grep -ohE 'https://[^) >]+' | sort -u`, then `curl -sIL --max-time 10` each -> a 404 or a domain gone is a finding; a transient failure is not.
4. Model ids and versions: the same grep for `claude-` -> a model id or tool version no longer current is a finding.
5. Pick at most three, in this order: broken link or contradicted claim -> stale doc to delete (with its index rows) -> compression drift from the writing rules in `CLAUDE.md`.
6. Ship one small PR per improvement, its evidence in the body, `bash bin/check` green first. fail -> stop: do nothing rather than open an empty PR.
