# 🎯 Opportunistic Improvement — take the opportunity, inside the limits

**A cost already paid is the moment to improve what rides it.** Downtime, an open system, a restart, a migration window — the expensive part is done once; every change that can ride it for free should. The limits are fixed: only what has a switch back and a gate, never a one-way door, never extra risk on a cutover.

Shipping is the value: an ask is done when it is merged, deployed and verified, and it leaves the project better than it found it. Proactive, scoped → [../writing-for-agents/behavioral-rules.md](../writing-for-agents/behavioral-rules.md#-proactive--but-about-the-problem-not-the-diff).

## ✅ Take it — the rules (as of 2026-11)

| # | Rule | Because |
|---|---|---|
| 1 | **Name the non-negotiable first** — no data loss, working backups. Everything else is negotiable | a plan that never states the hard line trades it away under time pressure |
| 2 | **When a cost is already paid, fold in every change that rides it** | the next window costs the same downtime again, for one change |
| 3 | **Fold in only what has a switch back and a pass/fail gate** | an add-on without either turns one failure into two you cannot tell apart |
| 4 | **Fix the latent defect a change would widen first, in its own PR** | more traffic, more tenants or a new path through old code turns a quiet bug into an incident |
| 5 | **Fix a dependency upstream, release it, pin the released artifact** | a local patch or a fork is a second copy nobody updates; a pinned release is one fact |
| 6 | **Latest minor inside the window, never a new major** | a minor ships fixes under the same contract; a major is a migration of its own |
| 7 | **Never fold in schema changes, id schemes, renames or refactors of money code** — write each down with a trip condition | each is a [one-way door](shipping-doctrine.md#replace-dont-accumulate) or touches what cannot be rolled back; it earns its own change, gate and record → [decision-records.md](decision-records.md) |
| 8 | **Merge infrastructure inert** (0 replicas, no route), activate it with small pin PRs | the big PR proves it builds and deploys; each activation is a one-line change you can read and undo |
| 9 | **Waiting time is for the next slice, on disjoint paths** | CI and a rollout are idle minutes; a slice on other files spends them without a merge conflict |
| 10 | **One gated large PR before go-live; one small PR per fix after** | before go-live nothing depends on it, so one review covers it; after, each fix must be readable alone |
| 11 | **Prove before each irreversible step** | past it, a wrong assumption costs data, not a retry → [data-risk-ladder.md](data-risk-ladder.md) |
| 12 | **Past the point of no return, tear down the old system fast — keep the data copy** | two live systems drift and split writes; a data copy is the only rollback that still works |
| 13 | **Every surprise becomes a recorded lesson or a script during the run** | written later it is forgotten; the next run hits the same surprise |
| 14 | **Never add risk to a cutover for developer-experience work** | a cutover failing over a nicer log format is the worst trade on the table; DX waits for a quiet change |

## 🧭 Before executing: name what would make it markedly better

| Do | Then |
|---|---|
| Name the better tool, the upstream fix, the trick an elite engineer knows — not the workaround | take it when rules 2–3 hold and rules 7 and 14 do not forbid it |
| Check how the existing system behaves before designing a constraint for it | the constraint matches reality, not a guess |
| Out of scope, too risky, or a one-way door | **file it** — an issue with the trigger and the evidence. Filed is not dropped; silently skipped is |

## 🚫 What this is not

| Not | Because |
|---|---|
| A license to widen a diff | every folded-in change still traces to a paid cost and passes the same gate → [shipping-doctrine.md](shipping-doctrine.md) |
| A reason to skip a gate | a gate failing on a wanted change means fix it and re-gate, never drop the gate |
| Taste | the bar is a named benefit and a named switch back |

---

**Related:** [data-risk-ladder.md](data-risk-ladder.md) — the ladder every data-touching step climbs · [bug-sweep.md](bug-sweep.md) — finding defects on a cadence · [../architecture/data-and-scale.md](../architecture/data-and-scale.md) — shape now, capacity later
