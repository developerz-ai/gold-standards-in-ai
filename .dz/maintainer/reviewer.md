# Reviewer instructions — developerz-ai/gold-standards-in-ai

> Read by the developerz.ai reviewer at EVERY review of this repository, at the reviewed
> commit, as this repo's own layer of the reviewer's prompt — established by dz-runner setup mode.
> It refines the platform rules (a finding is cited or withheld; the bot always discloses)
> and never overrides them. Edit freely; keep it under 12,000 characters — past that it is cut.

## Stack

Markdown only (no runtime). The gate is `bin/check` (`[ -d docs ]||exit 1; o=$(set +e; find . -name '*.md' | while read -r f; do d=${f%/*}; b=${f##*/}; case $f in ./docs/*) [ $b = README.md ] || grep -qsF $b README.md $d/README.md || echo $f;; esac; sed '/^ *```/,/^ *```/d;s/`[^`]*`//g' "$f" | grep -oP '\]\(\K[^)#[:space:]]+' | grep -v ':' | while read -r l; do [ -e "$d/$l" ] || echo "$f $l"; done; done;true); [ -z "$o" ]||{ echo "$o"; false; }`): a change that does not pass it is not proven, whatever its description says.

## Review conventions

- TODO(setup): the conventions a reviewer must hold this repo to (from CLAUDE.md / CONTRIBUTING), each as a rule.

## Gates that matter

- TODO(setup): which checks decide a change here, and what "proven" means for a pull request.

## Never flag

- TODO(setup): what never to flag — generated or vendored files, style a formatter owns, conventions this repo chose on purpose.
