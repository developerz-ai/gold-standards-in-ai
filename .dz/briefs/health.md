# Brief: is the project healthy?

Daily checkup fired by `schedules: health`: the failures no issue or PR announces.

| Signal | Command | Healthy |
|---|---|---|
| every relative link resolves | the loop below | it prints nothing |
| every doc is indexed | `for f in $(git ls-files 'docs/**/*.md'); do b=$(basename "$f"); [ "$b" = README.md ] \|\| grep -rqF "$b" README.md "$(dirname "$f")/README.md" \|\| echo "$f"; done` | it prints nothing |
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

## What to file

For each unhealthy signal, one ask, unless an open issue already carries its exact title
(`gh issue list --state open --search "<title> in:title" --json number,title`, then compare byte for byte).
Titles are stable: nothing that changes run to run (sha, run URL, version) goes in one; that goes in the body,
with the command and its output. Unreadable evidence is no verdict: re-run it, never file it as a defect.
A red `main` comes first: nothing releases from it. A stuck Dependabot PR comes next.

## Rows

- `relative-links` — Every relative link resolves; use the existing relative-link loop above.
- `docs-indexed` — Every doc is indexed; use the existing doc-indexing command in the Signal table above.
- `main-ci` — The newest completed `check` run is `success` at the sha `main` names now; a finding carries the run URL, the head sha and `main`'s current sha; a red run on a superseded sha and an errored `gh` call are not findings (re-run, never file); use the existing default-branch CI command in the Signal table above.
- `dependabot-prs` — No open Dependabot PR is older than 7 days, one `.github/dependabot.yml` schedule interval; a finding carries the PR number, its age in days and its URL; a younger PR is normally progressing and an errored `gh` call is missing evidence (re-run, never file); use the existing Dependabot PR command in the Signal table above.
