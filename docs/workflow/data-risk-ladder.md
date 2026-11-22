# 🪜 Data-Risk Ladder — never lose data

**Any step that can lose data climbs the ladder, one rung at a time, each rung proven before the next.** A migration, a cutover, a datastore move, a restore, a teardown. Losing data is the one failure an agent cannot fix forward.

## 🪜 The rungs (as of 2026-11)

| # | Rung | Proof before the next rung |
|---|---|---|
| 1 | **Stop the writers** — scale them to zero, close the queue, set the source read-only | the process list or replica count shows none running |
| 2 | **Prove zero writes** | a write counter, the WAL position or a row count read twice, minutes apart, unchanged |
| 3 | **Back up** | the backup exists in a second place, with a size and a timestamp after rung 2 |
| 4 | **Verify the backup by restoring it** | restored into a scratch target; row counts and a checksum or spot query match the source |
| 5 | **Switch** — point the readers and writers at the new target | a live probe reads and writes through the new path; the old one takes no traffic |

A backup nobody restored is a hope, not a backup. Rung 4 is the one skipped under time pressure, and the one that matters.

## ♻️ After the switch

| Rule | Because |
|---|---|
| **Tear down the old system fast** | two live systems drift and split writes |
| **Keep the old data copy until its retention holds** | it is the only rollback left past the point of no return |
| **Delete the last copy only as a human act** | nothing restores it → [the acts that stay human](../00-philosophy.md#who-this-is-for-ai-writes-100-of-the-code) |
| **Never rotate a live secret as a side effect** | every holder of the old key breaks at once, mid-cutover |

## 🔁 Fix forward, never revert

A broken change is fixed by a new change: a red deploy gets a fix PR, written from the evidence — logs, the failing probe, the trace.

| Why not revert | The mechanism |
|---|---|
| Migrations do not revert | a migration runs once, everywhere, on merge → [merge → deploy](../developer-experience/ai-first-cicd.md#6-merge--deploy-without-a-human-bottleneck); reverting the code leaves old code on the new schema |
| A revert drops what rode the change | the other fixes in the PR go with it, and the next attempt starts from less |
| A revert is an unplanned change of state | it skips the thinking the fix needed; the defect comes back in the re-land |

Data written by the bad change is repaired forward too: a backfill with its own gate → [../architecture/data-and-scale.md](../architecture/data-and-scale.md#-migrations-vs-backfills).

---

**Related:** [opportunistic-improvement.md](opportunistic-improvement.md) — what may ride a cutover, and what never does · [shipping-doctrine.md](shipping-doctrine.md) — one path, replaced completely
