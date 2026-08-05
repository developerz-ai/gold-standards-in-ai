# 🚢 Shipping Doctrine — no legacy, no shims, no flags

Agents ship fast. Without a doctrine, "fast" becomes a codebase full of half-replaced paths, dead flags, and shims nobody dares delete — and every one of those doubles the states the next agent has to reason about.

**One path, replaced completely, in the same PR.**

## Replace, don't accumulate

| ❌ Accumulating | ✅ Replacing |
|---|---|
| dual read/write paths "for safety" | one write path; the migration + backfill ship in the same change |
| a deprecated shim kept "for one release" | callers updated in the same PR |
| `v2` beside `v1` in an internal API | one version; internal callers move with it |
| a compatibility branch on a config value | the config is migrated, the branch is deleted |

Rule of thumb: **if you would be embarrassed to explain the second path to a new agent, don't create it.** The agent has no memory of why it exists; it will read both as intentional.

Public, external contracts are the genuine exception — versioning an API other people call is not a shim. **Name the one-way doors** (a stored data shape, a promise to users, a public contract, an id scheme) and reason about them before shipping. Everything else can be revised later; don't pay for it today.

## 🚫 Never feature flags

Rejected, not deferred. Flags hide untested combinations — N flags means 2^N states nobody has ever run — and people forget them, so the flag becomes permanent config with no owner.

What to use instead:

| Want | Use |
|---|---|
| per-tenant behavior | an org/tenant **config record** (KV), read at the seam, with a default |
| spend/limit differences | quota overrides |
| a risky rollout | ship it off, then turn it on by config for one tenant, then default it — and **delete the branch** |
| "we might want to revert" | a small PR and `git revert` |

The difference between a flag and a config record: the config record is **data with a schema and an owner**, visible in the admin surface. The flag is code with neither.

## 🚫 Never A/B in production

Also rejected — specifically for anything an agent or model touches.

- Each arm doubles untested combinations.
- An arm changes what the **model** sees: a bug report can't be reproduced without knowing which arm served it, and a handed-off session can flip arms mid-answer.
- Pick before merge, ship one path. **Eval comparison before merge is not an A/B** — that's the right way to choose.

## Ship the guard with the fix

Every PR that fixes a defect that *could recur* also ships its mechanical guard — a lint rule, a contract test, an assertion at the seam. Fix the pattern, not the instance → [../writing-for-agents/guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md).

And if it felt bad, automate it **in the same PR**: the same three commands again, prints added then deleted, setup knowledge in one head, a command that answers with silence instead of an error. Don't tell the next person to be careful.

## Never ship a fix you know is wrong

Fast path ≠ right path → say so in one sentence and take the right one. That is the default, not an escalation. Refuse specifically:

- patching a symptom while the cause survives;
- silencing a failing test, a type error, or a lint guard to get green — **the guard is the asset**.

**Small and wrong are different axes.** The minimum change that solves the problem *correctly* is still the goal; "surgical" is never a license to gold-plate, and never a license to leave the cause alive.

If the quick fix is chosen anyway, build it properly *within* that choice and make the debt impossible to lose: a test pinning the known-bad behavior, or a `docs/gotchas.md` line. Never a bare `TODO`.

## When the same bug keeps coming back, the design is the bug

Signals, not bad luck: a recurring defect class · a fix only makeable by editing three files · an invariant with no single seam · a comment asserting a guarantee the code no longer provides.

Then **the refactor is the fix** — but scope it and propose it as **its own change**, never smuggled into the bug fix. Sunk cost is not an argument. The bar is evidence you can name, not taste.

## Be stubborn about consequences, not preferences

Calibrate resistance to **blast radius and reversibility**, never to how many times the ask was repeated.

- Reversible and low-stakes → state the view once, then defer and build it well.
- **Backfires later** — silent data loss, an irreversible migration or backfill, a disabled guard, a "temporary" shim, an id scheme you can't walk back — name the failure, when it surfaces, and what it costs *then*.
- Insisting harder is not new information. It becomes a decision when it engages with the named consequence.
- Proceeding anyway → leave evidence (a pinning test, a gotcha line).

## Definition of done

**Done means deployed and verified.** A green local gate is not done. An open PR is not done. A merged PR whose roll you didn't confirm is not done. Report what you *verified* → [../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md#6-merge--deploy-without-a-human-bottleneck).

---

**Related:** [../writing-for-agents/workflow-commands.md](../writing-for-agents/workflow-commands.md) — `/feature`, PR sizing · [../architecture/data-and-scale.md](../architecture/data-and-scale.md) — migrations & backfills ship with the code
