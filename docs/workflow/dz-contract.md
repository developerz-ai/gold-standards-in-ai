# 📜 The `.dz/` Contract

A repo the maintainer bot keeps carries one directory, `.dz/`, that tells the bot what it may do, how to review, and what to check on a schedule. Every file in it is read by a machine with a parser and a cap. **Pin the contract with a CI check, or it drifts silently:** an over-cap brief is refused every tick, a brief with no rows files nothing, and a setup stub reaches the reviewer as an instruction. As of 2026-11; facts from the policy schema source.

## Layout

| Path | Holds | Read |
|---|---|---|
| `.dz/maintainer/maintainer.yml` | the policy — what the bot may do here | first; the root `.maintainer.yml` is a legacy home, read **only** when the `.dz/` file is absent — never merged with it |
| `.dz/maintainer/reviewer.md` | the repo's own reviewer prompt, one layer of the reviewer's instructions | at the **base branch tip**, never the reviewed head — a PR cannot rewrite the rules it is judged by; legacy home `.claude/agents/reviewer.md` |
| `.dz/schedules/<name>.md` | one brief per scheduled checkup | when its schedule fires, at the default branch's HEAD; any repo-relative `brief_file` works, so pick ONE home per repo |
| `.dz/onboarding/scorecard.md` | what onboarding proved about the repo, item by item, with evidence | written by onboarding; humans and agents read it |
| `.dz/README.md` | what lives here and who reads it | humans |

Stays at the repo root, because the tools that read them look nowhere else: `.mcp.json` ([../writing-for-agents/mcp-json.md](../writing-for-agents/mcp-json.md)), a third-party reviewer's config.

## The policy file

```yaml
# yaml-language-server: $schema=<the published maintainer.v1.json URL>
version: 1          # required, the only required key; every block omitted takes its default

pr:
  required_checks: ["check"]   # the raw check-run name, never the UI's "check / check"
  auto_merge: true

schedules:
  - name: health               # unique in the file; keys the firing, its tasks, its audit rows
    days: [mon, thu]           # non-empty subset of mon..sun, UTC
    hours_utc: [6]             # non-empty, whole hours 0-23
    # weeks_of_month: [1, 3]   # optional, 1-4; week n = UTC days 7n-6..7n. Omit to fire every week
    brief_file: .dz/schedules/health.md
    # repos: [owner/other]     # optional, at most 4 more; never the repo holding this file
```

| Rule | Mechanism |
|---|---|
| Unknown key → refused by name, with the nearest known key | a typo (`automerge`) once parsed `ok` and the written opt-out never existed; only keys the platform deleted are dropped silently |
| `schedules[]` is a closed shape: `name`, `days`, `hours_utc`, `weeks_of_month`, `brief_file`, `repos` | no `cron:` — a typo'd cron fires every minute and each firing holds a box on the tenant's key |
| At most 10 schedules | every entry that comes due holds one of the account's boxes for its whole run |
| Fortnightly or monthly goes in `weeks_of_month`, never in the brief | a brief that says "act only in the first week" still fires, and bills, every other week → [maintainer-token-spend](../ai-agents/maintainer-token-spend.md#-cadence--weekly-fortnightly-monthly) |
| `name`, `days`, `hours_utc`, `weeks_of_month`, `repos` unique | checked semantically, after the shape |

## Briefs — `## Rows` and the size cap

A checkup files **only a failing row its brief declares**, one ask per row. The brief declares rows like this:

```markdown
## Rows

- `main-green`: the newest completed run on main passed; a finding carries the run URL.
- `docs-indexed`: every doc is linked from both indexes; use `bin/check-docs`.
```

| Rule | What breaks otherwise |
|---|---|
| One `## Rows` heading (`## 6. Rows` in a numbered brief); only the first is read | a second section is ignored |
| A row is a list item that **opens** with a backticked id: lowercase `[a-z0-9][a-z0-9_-]`, 1–40 chars | an item without the leading id, or `` `Main_Green` ``, is not a row |
| The section ends at the next heading; fenced code is never read | an example inside a fence is safe; a row under a later heading is not a row |
| A brief with no rows still fires | it files nothing, forever, and the bot opens one ask to add rows |
| **Cap: 10,000 chars for the brief plus its header** | over the cap the firing is refused on every tick — never cut, never partially run |

The header is one platform-written line prepended with a blank line after it, so the brief's real budget is smaller than the cap:

```
dz-checkup v1 repo=<owner/name> rows=<id>,<id>,... brief_file=<path>
```

`budget = 10000 − len(header) − 2`. A long repo name, many rows and a deep path each cost bytes. Keep briefs well under the cap — a brief that grows a row at a time crosses it without anyone touching the cap.

## No `TODO(setup)` left

Onboarding writes floors with `TODO(setup)` stubs (the reviewer prompt's sections, `CLAUDE.md` sections it could not fill). **Readers do not strip the marker**: an unfilled stub reaches the reviewer and the coder as an instruction. Fill the section from what the repo actually does, or delete it — an empty section is honest, a stub is not.

## Pin it in CI

One script, run by `bin/check` so the local gate and CI are the same command ([../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md)). Three checks: the policy's shape, every scheduled brief's rows and headed length, no setup stub.

```ts
// scripts/check-dz.ts — pins the repo's .dz/ contract. Exit 1 on any finding.
// Usage: GITHUB_REPOSITORY=owner/name bun run scripts/check-dz.ts   (dev dep: ajv@8)
import { existsSync, readdirSync, readFileSync, statSync } from "node:fs";
import { join } from "node:path";
import Ajv from "ajv";

const PROBLEM_MAX = 10_000; // brief + header line + blank line, in JS string length
const ROW_ID = /^[a-z0-9][a-z0-9_-]{0,39}$/;
const MARKER = "TODO(setup)";
const repo = process.env.GITHUB_REPOSITORY ?? "";
const errors: string[] = [];

if (!/^[^\s/]+\/[^\s/]+$/.test(repo)) throw new Error(`GITHUB_REPOSITORY=${JSON.stringify(repo)}: want owner/name`);

// 1. The policy: .dz/ home first, root file only when the .dz/ one is absent.
const policyPath = [".dz/maintainer/maintainer.yml", ".maintainer.yml"].find((p) => existsSync(p));
if (!policyPath) throw new Error("no policy at .dz/maintainer/maintainer.yml or .maintainer.yml");
const text = readFileSync(policyPath, "utf8");
const policy = Bun.YAML.parse(text) as { schema?: string; schedules?: { name: string; brief_file: string }[] };
const schemaUrl = policy.schema ?? /\$schema=(\S+)/.exec(text)?.[1];
if (!schemaUrl) throw new Error(`${policyPath}: name the JSON schema (a schema: key or a yaml-language-server $schema= comment)`);
const schema = await (await fetch(schemaUrl)).json();
const validate = new Ajv({ strict: false, allErrors: true, validateFormats: false }).compile(schema);
if (!validate(policy)) for (const e of validate.errors ?? []) errors.push(`${policyPath}: ${e.instancePath || "/"} ${e.message}`);

// 2. Every scheduled brief: present, declares ## Rows, fits the cap WITH its header.
function rowsOf(brief: string): string[] {
  const rows: string[] = [];
  let fenced = false, inRows = false;
  for (const line of brief.split(/\r?\n/)) {
    if (/^\s{0,3}(```|~~~)/.test(line)) { fenced = !fenced; continue; }
    if (fenced) continue;
    if (/^##\s+(?:\d+\.\s+)?rows\s*$/i.test(line)) { if (inRows) break; inRows = true; continue; } // a second Rows heading ends the section
    if (/^#{1,6}\s/.test(line)) { if (inRows) break; continue; }
    const id = inRows ? /^\s{0,3}[-*+]\s+`([^`]+)`/.exec(line)?.[1] : undefined;
    if (id && ROW_ID.test(id) && !rows.includes(id)) rows.push(id);
  }
  return rows;
}
for (const s of policy.schedules ?? []) {
  if (!existsSync(s.brief_file)) { errors.push(`schedule ${s.name}: brief_file ${s.brief_file} does not exist`); continue; }
  const brief = readFileSync(s.brief_file, "utf8");
  const rows = rowsOf(brief);
  if (rows.length === 0) errors.push(`${s.brief_file}: no \`## Rows\` items — the checkup fires and files nothing`);
  const header = `dz-checkup v1 repo=${repo} rows=${rows.join(",")} brief_file=${s.brief_file}`;
  const chars = header.length + 2 + brief.length;
  if (chars > PROBLEM_MAX) errors.push(`${s.brief_file}: ${chars} chars with header > ${PROBLEM_MAX} — refused on every tick; cut ${chars - PROBLEM_MAX}`);
}

// 3. No setup stub left where an agent reads it as an instruction.
function walk(dir: string): string[] {
  return readdirSync(dir).flatMap((n) => {
    const p = join(dir, n);
    return statSync(p).isDirectory() ? walk(p) : [p];
  });
}
for (const f of [...(existsSync(".dz") ? walk(".dz") : []), "CLAUDE.md"].filter(existsSync)) {
  readFileSync(f, "utf8").split("\n").forEach((line, i) => {
    if (line.includes(MARKER)) errors.push(`${f}:${i + 1}: ${MARKER} left — fill the section or delete it`);
  });
}

for (const e of errors) console.error(e);
process.exit(errors.length ? 1 : 0);
```

| Limit of this check | Close it with |
|---|---|
| The published JSON schema checks **shape only** — most blocks accept unknown keys there, so a typo passes | the platform's authenticated policy-validate endpoint (parse + shape + semantic), called with a repo-write token from a CI secret |
| Duplicate schedule names, days or hours pass the JSON schema | same endpoint |
| A brief's rows can be well-formed and still unanswerable | review the brief like code: each row names its command and what a finding carries |

**Observe it red** before trusting it: plant an over-cap brief, rename `## Rows`, add a `TODO(setup)` line, set `version: 2` — each must print its own line and exit 1 ([../writing-for-agents/guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md)).

## What the policy cannot express yet

Write these in prose where the agent reads them — not as invented keys, which the parser refuses.

| Want | Schema today | Write it in |
|---|---|---|
| A PR size cap / one PR in flight | only `pr.large_threshold_lines` — a hold-for-a-human on merge, not a bound on what the agent produces | `CLAUDE.md` ([../writing-for-agents/workflow-commands.md](../writing-for-agents/workflow-commands.md)) |
| A performance / N+1 / leak review lens | review passes are `correctness`, `security`, `tests`, `api_contract`, `style_nits` | `.dz/maintainer/reviewer.md` — including the opposite ("do not flag N+1 here, preloading is global") |
| Where specs, plans and standards live | no key | `CLAUDE.md`, an index into `docs/` ([../writing-for-agents/claude-md.md](../writing-for-agents/claude-md.md)) |
| Services beyond the closed `services` vocabulary (`redis`, `postgres`) | closed enum, on purpose — a free-form value would start arbitrary processes | the repo's own `bin/setup` / `services: {up, down}` commands |
| A test that pins `.dz/` | none on the platform side | the check above |

---

**Related:** [bug-sweep.md](bug-sweep.md) — the scheduled sweep a checkup brief drives · [project-from-zero.md](project-from-zero.md) — `.dz/` lands with the agent tooling · [../ai-agents/untrusted-input.md](../ai-agents/untrusted-input.md) — repo config is executable surface
