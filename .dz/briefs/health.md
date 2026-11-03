# Brief: is the project healthy?

Daily checkup fired by `schedules: health`: the failures no issue or PR announces.

| Signal | Command | Healthy |
|---|---|---|
| every relative link resolves | the loop below | it prints nothing |
| every doc is indexed | `for f in $(git ls-files 'docs/**/*.md'); do b=$(basename "$f"); d=$(dirname "$f"); [ "$b" = README.md ] \|\| { grep -rqF "$b" README.md && grep -rqF "$b" "$d/README.md"; } \|\| echo "$f"; done` | it prints nothing |
```bash
git ls-files '*.md' | grep -v '^\.claude/' | while read -r f; do
  awk '/^[[:space:]]*```/{c=!c;next} !c' "$f" | sed 's/`[^`]*`//g' \
    | grep -oE '\]\([^)]*\)' | sed 's/^](//;s/)$//' | grep -vE '^[a-z]+:' | while read -r l; do
      case $l in
        \#*) t=$f; g=${l#\#} ;;
        *)    g=${l#*#}; [ "$g" = "$l" ] && g=; t="$(dirname "$f")/${l%%#*}" ;;
      esac
      if [ ! -e "$t" ]; then echo "$f -> ${t#$(dirname "$f")/}"; continue; fi
      [ -n "$g" ] || continue
      { awk '/^[[:space:]]*```/{c=!c;next} !c' "$t" | sed -n 's/^#\{1,6\}[[:space:]]*//p' \
          | tr '[:upper:]' '[:lower:]' | sed -e 's/ /-/g' -e 's/[^[:alnum:]_-]//g'
        awk '/^[[:space:]]*```/{c=!c;next} !c' "$t" \
          | grep -oE '<a[[:space:]]+(name|id)="[^"]+"' | sed 's/.*="//;s/"$//'
      } | grep -qxF -- "$g" || echo "$f -> $l dead anchor"
    done
done
```

It skips fenced and inline code, where a link is an example rather than a link, and `.claude/`.
It resolves a `#fragment` against the target file's headings — the slug of the heading text: lowercased, every character that is not a letter, a digit, `-` or `_` deleted, spaces turned into `-` — and files a fragment that matches no heading. Emoji, arrows and dashes are deleted, not turned into hyphens; a literal `-` and `_` are kept, so `## ⌨️ Keyboard paths` is `-keyboard-paths` and `## 6. Merge → deploy without a human bottleneck` is `6-merge--deploy-without-a-human-bottleneck`. A missing target file is filed once, for the file, without its fragment.

This is the same slug `bin/check-docs` computes — the two must not drift, or the brief files dead anchors on a green tree.

## What to file

For each unhealthy signal, one ask, unless an open issue already carries its exact title
(`gh issue list --state open --search "<title> in:title" --json number,title`, then compare byte for byte).
Titles are stable: nothing that changes run to run (sha, run URL, version) goes in one; that goes in the body,
with the command and its output. Unreadable evidence is no verdict: re-run it, never file it as a defect.
A red `main` comes first: nothing releases from it.

## Rows

- `relative-links` — Every relative link resolves; use the existing relative-link loop above.
- `docs-indexed` — Every doc is indexed; use the existing doc-indexing command in the Signal table above.
