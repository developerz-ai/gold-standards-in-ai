# Brief: is the project healthy?

Daily checkup fired by `schedules: health`: the failures no issue or PR announces.

| Signal | Command | Healthy |
|---|---|---|
| every relative link resolves | the loop below | it prints nothing (tracked files only — narrower than the gate) |
| every doc is indexed | `for f in $(git ls-files 'docs/**/*.md'); do b=$(basename "$f"); [ "$b" = README.md ] \|\| { grep -qF "$f" README.md \|\| echo "$f (root README)"; grep -qF "$b" "$(dirname "$f")/README.md" \|\| echo "$f (section README)"; }; done` | it prints nothing; the probe requires both required indexes and is stricter than `bin/check-docs` on index coverage |
| red default-branch CI | `gh run list --branch main --workflow check --limit 5 --json headSha,conclusion,status,url` | the newest completed run is `success` at the sha `main` names now |
| stuck Dependabot PR | `gh pr list --state open --author app/dependabot --json number,title,createdAt,url` | no open PR is older than 7 days, one `.github/dependabot.yml` schedule interval |
```bash
git ls-files '*.md' | grep -v '^\.claude/' | while read -r f; do
  awk '/^[[:space:]]*```/{c=!c;next} !c' "$f" | sed 's/`[^`]*`//g' \
    | grep -oE '\]\([^)#[:space:]]+' | sed 's/^](//' | grep -vE '^[a-z]+:' | while read -r l; do
      [ -e "$(dirname "$f")/$l" ] || echo "$f -> $l"
    done
done
```

It skips fenced and inline code, where a link is an example rather than a link, and `.claude/`.

Both probes read tracked files only (`git ls-files`). The gate, `bin/check` = `bin/check-docs` + `bin/test`, also walks
untracked Markdown and `.claude/` Markdown, and skips any link target containing a colon — the loop above skips only a
target starting with a lowercase scheme. The doc-indexing probe requires both required indexes and is stricter than
`bin/check-docs` on index coverage: a probe finding is a real one-index gap the gate misses. `bash bin/check` remains
the map verdict.

## What to file

For each unhealthy signal, one ask, unless an open issue already carries its exact title
(`gh issue list --state open --search "<title> in:title" --json number,title`, then compare byte for byte).
Titles are stable: nothing that changes run to run (sha, run URL, version) goes in one; that goes in the body,
with the command and its output. Unreadable evidence is no verdict: re-run it, never file it as a defect.
A red `main` comes first: nothing releases from it. A stuck Dependabot PR comes next.

## Rows

- `relative-links` — Every relative link resolves; use the existing relative-link loop above.
- `docs-indexed` — Every doc is indexed in **both** required indexes: the root `README.md` and the section `README.md` next to the doc (for a top-level doc such as `docs/00-philosophy.md`, that section index is `docs/README.md`). Use the existing doc-indexing command in the Signal table above; it prints one line per missing index — `… (root README)` or `… (section README)` — so a doc indexed in only one of the two is reported. One ask per missing index, not one per doc. This probe is stricter than `bin/check-docs` on index coverage; `bash bin/check` remains the map verdict.
- `main-ci` — The newest completed `check` run is `success` at the sha `main` names now; a finding carries the run URL, the head sha and `main`'s current sha; a red run on a superseded sha and an errored `gh` call are not findings (re-run, never file); use the existing default-branch CI command in the Signal table above.
- `dependabot-prs` — No open Dependabot PR is older than 7 days, one `.github/dependabot.yml` schedule interval; a finding carries the PR number, its age in days and its URL; a younger PR is normally progressing and an errored `gh` call is missing evidence (re-run, never file); use the existing Dependabot PR command in the Signal table above.
