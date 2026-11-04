# Brief: is the project healthy?

Daily checkup fired by `schedules: health`: the failures no issue or PR announces.

| Signal | Command | Healthy |
|---|---|---|
| every relative link resolves | `bin/check-docs` | it prints nothing — fences and inline code are skipped, so a printed link is a real one |
| every doc is indexed | `bin/check-docs` | it prints nothing; a doc is indexed only in **both** required indexes — the root `README.md` and its section `README.md` |
| red default-branch CI | `gh run list --branch main --workflow check --limit 5 --json headSha,conclusion,status,url` | the newest completed run is `success` at the sha `main` names now |
| stuck Dependabot PR | `gh pr list --state open --author app/dependabot --json number,title,createdAt,url` | no open PR is older than 7 days, one `.github/dependabot.yml` schedule interval |

One validator: `bin/check-docs` — the gate and this checkup run the same code, so a finding here is a finding there.
The checkup walks untracked Markdown and `.claude/` Markdown too, skips fenced and inline code — a link there is an
example, not a link — and never reports a target containing a colon; a printed finding is a real one.
`bash bin/check` (`bin/check-docs` + `bin/test`) remains the map verdict.

## What to file

For each unhealthy signal, one ask, unless an open issue already carries its exact title
(`gh issue list --state open --search "<title> in:title" --json number,title`, then compare byte for byte).
Titles are stable: nothing that changes run to run (sha, run URL, version) goes in one; that goes in the body,
with the command and its output. Unreadable evidence is no verdict: re-run it, never file it as a defect.
A red `main` comes first: nothing releases from it. A stuck Dependabot PR comes next.

## Rows

- `relative-links` — Every relative link resolves; use `bin/check-docs`.
- `docs-indexed` — Every doc is indexed in **both** required indexes: the root `README.md` and the section `README.md` next to the doc (for a top-level doc such as `docs/00-philosophy.md`, that section index is `docs/README.md`). Use `bin/check-docs`; it prints one line per doc that misses either index, and one ask goes to each missing index.
- `main-ci` — The newest completed `check` run is `success` at the sha `main` names now; a finding carries the run URL, the head sha and `main`'s current sha; a red run on a superseded sha and an errored `gh` call are not findings (re-run, never file); use the existing default-branch CI command in the Signal table above.
- `dependabot-prs` — No open Dependabot PR is older than 7 days, one `.github/dependabot.yml` schedule interval; a finding carries the PR number, its age in days and its URL; a younger PR is normally progressing and an errored `gh` call is missing evidence (re-run, never file); use the existing Dependabot PR command in the Signal table above.
