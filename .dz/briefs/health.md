# Brief: is the project healthy?

Daily checkup fired by `schedules: health`: the failures no issue or PR announces.

| Signal | Command | Healthy |
|---|---|---|
| every relative link resolves | the loop below | it prints nothing |
| every doc is indexed | `for f in $(git ls-files 'docs/**/*.md'); do b=$(basename "$f"); [ "$b" = README.md ] \|\| grep -rqF "$b" README.md "$(dirname "$f")/README.md" \|\| echo "$f"; done` | it prints nothing |
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
A red `main` comes first: nothing releases from it.

## Rows

- `relative-links` — Every relative link resolves; use the existing relative-link loop above.
- `docs-indexed` — Every doc is indexed; use the existing doc-indexing command in the Signal table above.
