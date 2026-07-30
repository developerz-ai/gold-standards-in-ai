---
description: End-to-end doc workflow for gold-standards-in-ai — scope, distrust the existing map, explore in parallel, split into path-disjoint slices, write with a hive of agents in this one checkout (no worktrees), gate on house rules + link integrity + the map, PR, merge. Markdown only; no build, no tests.
argument-hint: <the doc or topic you want written/revised> [+ reference URL(s)]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, SendMessage, TaskCreate, TaskUpdate, TaskList, Skill, WebFetch
---

# /feature

Writer on **Gold Standards in AI** — the knowledge base agents read before starting a project. Markdown only. Read [`CLAUDE.md`](../../CLAUDE.md) and `docs/00-philosophy.md` first.

**Done means merged and the map true — nothing less counts.** No build, no tests, no deploy: the arc is understand → distrust the current map → explore → slice → write → **gate** (house rules · every link resolves · every doc on both READMEs) → PR → **merged**. A doc not wired into `README.md` and its section `README.md` does not exist. Report which of those you actually checked, not which you assume.

## Request
$ARGUMENTS

**The prompt is the context — read the intent.** Autonomy, sweep width, whether to confirm before merging: infer it from the words. "Just ship it" → run start-to-finish, decide everything, merge on a green gate, surface decisions in the PR body instead of asking. A tentative ask → clarify the genuinely ambiguous topic boundary and let the user review before merge. The flow below is a map, not a checklist to recite. Always stop for a real blocker: a load-bearing claim you cannot source, a duplicate that needs a *delete* decision, anything leaking secrets, private hostnames/IPs or client names.

**Pick the PR mode before you brief anyone.** Slice-per-PR (default, one topic or section each) vs one fat PR ("do it in 1 PR" — legitimate for a coherent section rewrite). Path-disjointness still governs the *writing* either way; it just stops governing the *commit*.

**Cap a PR at ~25–30 changed files** — lower than a code repo on purpose: nobody proofreads 100 docs. Past that review is a formality, one disputed claim holds the set hostage, and a later "when did this go stale?" bisect lands on one enormous commit. Split even if the user asked for one PR, and say why — the agent boundaries were disjoint by construction, so each is already a PR. Land shared vocabulary (a new section `README.md`, a term everything links to) **first**, then the docs citing it.

## Work as a hive mind, in one checkout

**Hiving is a judgement call, not a ritual.** Two things justify it: **searching** (a broad sweep across `docs/**` where you want conclusions, not file dumps) and **scale** (several independent docs, separable by path). Nothing else. One doc, one paragraph, a correction you already understand — write it yourself; briefing + collision management + report-reading costs more than the edit, and you pay it in the one context that must survive to the merge.

**Never use git worktrees** — no `isolation: worktree` on the `Agent` tool, no `git worktree add`, no per-agent directories, ever. Nothing to install here, which makes the real cost clearer rather than smaller: a second tree hides half-written docs from the link gate and from `git status`, and **`README.md` is a single shared file every slice must eventually touch** — editable only in the tree everyone else is in. One checkout, many hands; the file set is the only lock.

- **You coordinate; you do not write.** You own git, the ledger and the merge, and you are the only participant who must survive to the end — spend that context on routing, not on reading docs an agent will report back. Drafting prose yourself means you took a slice from someone who had room for it.
- **The file set is the lock.** Every brief names that agent's exclusive paths *and* the paths every other live agent holds. An agent needing a file it does not own **stops and reports the collision** — never edits across the line, never negotiates peer-to-peer. You mediate: hand it to the owner, or re-cut the boundary.
- **The README index rows are the one shared choke point.** `README.md` and each section `README.md` are edited **last, by you**, after the docs land. Never hand two agents the same index file.
- **Agents are long-lived teammates.** New work in an area someone holds goes to them via `SendMessage` — they keep their context and their file lock. A second agent on the same paths means two writers and a lost edit.
- **Work in waves; each wave re-tasks the next.** Explore → write → wire up. Do not plan wave 3 before wave 1 reports; it will be wrong.
- **Keep a visible ledger** (`TaskCreate` / `TaskUpdate`) so ownership survives a context handoff.
- **Expect the hive to contradict you.** A good agent reports "that doc already exists at `docs/x/y.md:40`" or "this claim is false as of 2026-07". Drop the premise. Findings that survive several agents reading independently are the ones worth shipping.

Full doctrine — the one this repo documents: [`docs/ai-agents/hive-mind.md`](../../docs/ai-agents/hive-mind.md).

### Who runs which checks

No linter, no test runner. The gate is **prose review, link integrity, the map staying true** — and it splits like any gate.

| | Agent (per doc it owns) | Coordinator (once, at the end) |
|---|---|---|
| house rules | re-reads **its own** doc against the rules below | spot-checks tone consistency across the set |
| links | verifies the links **it wrote** resolve | the repo-wide link sweep, once |
| the map | reports the row its doc needs | edits `README.md` + section `README.md` and runs the map check |
| secrets | greps its own doc | the repo-wide grep |

**An agent checks only its own files; repo-wide green is yours, once, at the end.** Never N agents sweeping all of `docs/**` — the reports overlap and each costs you a read. The two final-gate checks (they also match inside fenced blocks, so example paths are expected noise — judge the hits, don't count them):

```bash
# 1. every relative link resolves
grep -rnoE '\]\([^)h][^)]*\)' --include='*.md' . | sed -E 's/\]\(/|/; s/\)$//' |
while IFS='|' read -r src link; do
  f="${link%%#*}"; d=$(dirname "${src%%:*}")
  [ -e "$d/$f" ] || echo "DEAD  ${src%%:*} -> $link"
done

# 2. every doc is on the map, in both places
for f in $(git ls-files 'docs/*/*.md' | grep -v '/README.md$'); do
  grep -q "$f" README.md                                   || echo "UNMAPPED (root)    $f"
  grep -q "$(basename "$f")" "$(dirname "$f")/README.md"   || echo "UNMAPPED (section) $f"
done
```

A real hit from either is a merge blocker.

### Two things only the coordinator can do

- **Every slice you NAME, you must dispatch.** Briefs tell each agent which teammates are live on which paths — so a named-but-unlaunched slice makes agents dutifully defer work to someone who does not exist, and it vanishes. This really happened: three briefs referenced an "agent C" never spawned, and two finished agents left it a combined six items. Keep the roster and the dispatched set as **one list**; reconcile **before** reading reports.
- **Reserve an "unowned" bucket, and expect to fill it mid-run.** The real change often lands in a file no slice covers — `docs/00-philosophy.md`, a section `README.md`, a sibling doc the new one now contradicts. A homeless finding is the one most likely to be quietly dropped: when a report says "the fix is outside my set", **assign it immediately** rather than filing it.
- **Look for causal chains across reports.** Only you see all of them. Two docs drifting on the same term is one glossary problem, not two edits; "reads oddly" plus "duplicates it" is usually one merge. After the reports land, spend one pass asking "does A explain B?" — it changes what you write and what you can drop.

## The flow

1. **Scope.** One sentence: what an agent will be able to do after reading this. One topic per file. A duplicate of an existing doc → revise that doc instead of adding one; a stale doc → delete it rather than let it rot.

2. **Distrust the paperwork.** The `README.md` map is itself a claim, and it rots. Check it against `git ls-files` and `git log` before planning off it — a doc may already cover the topic under another name, a linked doc may be gone, a row may point at a file nobody has touched in months. Dated claims (`As of 2026-06`) are the likeliest to be wrong now. **State plainly which claims you falsified**, so nobody rewrites a doc that already exists.

3. **Explore (parallel).** Fan out `Agent` Explore agents (very thorough) over **disjoint** areas of `docs/**`: which section owns the topic · what already covers part of it (`file:line`) · new vs revise vs delete · which READMEs need a row · which siblings need a cross-link. Cited URLs → `WebFetch`; extract the *mechanism*, not the marketing. Require of every finding: `file:line`, a one-sentence problem statement, what a reader would get wrong, and the brief premises they **falsified**. **Protect your own context** — do not read what an agent will report back.

4. **Fold in live user reports as first-class findings.** Mid-run the user may paste a transcript where an agent followed this repo and did the wrong thing. That is *confirmed in the field* and routinely outranks anything found by reading. Fix the doc that caused the misreading, ranked above equal-severity read-only findings. If an in-flight agent owns that doc, extend its brief with `SendMessage` rather than spawning a second agent onto the same path.

5. **Track.** `gh issue list` before creating anything; this repo does not use Linear. For a single doc, the PR is enough.

6. **Build — branch first, then fan out.**

   ```bash
   git fetch origin && git status --short   # expect a clean tree
   git checkout -b docs/<slug>
   ```
   Do it now, while the tree is clean. Nobody should ever be writing into `main`.

   Then fix slice boundaries **before launching anyone**: **one doc per agent, disjoint paths**. Two agents that must edit one doc are one slice, not two. Every brief carries all nine — omitting any one is how a run goes wrong:
   - **its exclusive file set**, and never write outside it;
   - **which other agents are live, on which paths**, so a collision is *reported*, not silently resolved;
   - each finding with `file:line`, the problem, and what a reader would get wrong — plus **permission to drop any finding the repo contradicts** (that is the agent working correctly);
   - **evidence first, diagnosis second** — the symptom (the passage, the transcript, the dead link) first; your hypothesis explicitly labelled **unverified**, to confirm or kill before writing. Briefs leading with a confident diagnosis send agents to rewrite the wrong doc;
   - **the house rules** (below), verbatim, non-negotiable;
   - **self-contained and cross-linked** — relative links to related docs, no duplication of a sibling;
   - **checks narrowed to its OWN files** — never a repo-wide sweep;
   - **no git operations at all** — no branch, commit, checkout, stash. You own all git; work is left uncommitted;
   - **never tell an agent to "ask me" — it cannot.** A subagent has no channel to the user, so a question is a dead end: it blocks or guesses. Two legal moves instead: **decide and flag** (write the most defensible version, state the assumption, mark the passage so you can overwrite it) or **stop and report** with the evidence. Then *you* take it to the user and re-task with `SendMessage`.

7. **Write.** House rules, non-negotiable:

   - Lead with the rule, not the reason. Fragments over sentences. Tables over paragraphs.
   - File paths, commands, env vars and model IDs verbatim — compress prose only.
   - Dense and copy-paste oriented. Scannable in one pass. One topic per file.
   - Emojis in headings/READMEs for navigation; code blocks stay clean.
   - Cross-link related docs with relative links. Date load-bearing claims (`As of 2026-07`).
   - No meta-framing, no rhetoric, no trailing summary paragraph.
   - Vendor-neutral, reusable examples; prefer the latest Claude model IDs.

8. **Wire it in.** Add a link + a one-line hook to `README.md` **and** the section `README.md` — you do this yourself, last, after every agent has finished. A new section needs its own `README.md`.

9. **Gate.** Run both checks from §Who runs which checks, plus: house rules held · no secrets, private hostnames/IPs or client names · no undated load-bearing claim · no duplication of a sibling doc.

10. **Commit & merge.** Let every agent finish first — never commit while one is still writing. **Sweep the leftovers before you stage**: scratch notes, a draft at the repo root, an agent's own summary file. Agents create them and rarely clean up; they must not ship.

    ```bash
    git fetch origin                          # did main move?
    git add docs/<paths> README.md            # name the paths — never `git add -A`
    git status --short                        # then READ it
    git commit && git push -u origin HEAD
    gh pr create                              # Summary + the doc-map rows changed
    ```

    Naming paths on `git add` is all the selectivity needed. **Never `git stash`** — one global stack shared with every concurrent agent. **Main moves under you**: after `git fetch`, intersect files-changed-on-main with files-changed-locally; a real overlap (almost always `README.md`) is **three-way merged**, never taken wholesale — a naive overwrite drops main's rows silently, with no conflict marker. No CI here, so `gh pr merge --squash` once review is done. One PR in flight at a time; `git fetch` before the next slice.

11. **Leave the trail straight.** A doc your change invalidated gets updated in the same PR; a doc it made redundant gets **deleted** in the same PR — a wrong doc costs more than a missing one.

## Hard rules (from CLAUDE.md — non-negotiable)

The writing rules in step 7 are the house style and are not negotiable per-doc. On top of them: **no secrets, no private hostnames/IPs, no client names**; every doc linked from `README.md` *and* its section `README.md`; delete stale docs rather than let them rot. Never `--force`, never `git stash`, never a git worktree.

## Output

Report what shipped, and be equally explicit about what didn't.

```
Docs:      <files added / revised / deleted>
Map:       README.md + <section>/README.md updated   <yes/no>
Gate:      house rules <ok>  links <n dead>  map <n unmapped>  secrets <clean>
Deferred:  <n> — <what, and why not now>              [never omit this line]
Falsified: <map/doc claims that were wrong, now corrected>
PR:        #NNN <merged / open>
```
