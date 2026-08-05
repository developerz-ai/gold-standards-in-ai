# 🧠 Smart Change Detection & Caching

Every loop — local check, CI run, image build, rollout — should do work **proportional to what changed**, not to how big the repo is. That's one idea applied at four layers, and it's the difference between a 30-second gate and a 12-minute one at agent speed.

> **Detect what changed. Cache what didn't. Restart only what executes the change.**

## 1. One detector, three consumers

Write change-detection **once**, as a pure function of the changed-file list, and let the local gate, CI, and the deploy plan all read it.

```ts
// scripts/dev/affected.ts — no git, no fs, no process: a total function → trivially testable
export function computeAffected(changedFiles: string[]): { workspaces: string[]; rustApps: string[] } { … }
```

- **Rules as data, not scattered conditionals** — a table of workspaces, a table of "root manifests that force a full sweep", a package→dependents fan-out map.
- **Pin it against CI with a test.** The local twin and the CI `detect` job must agree, or "green locally" stops meaning anything → [ai-first-cicd](ai-first-cicd.md#the-parity-test).
- **Fan out through the real dependency graph**, not by directory guesswork: a change in `packages/db` must re-verify everything that imports it.
- **A TS-only change never triggers a Rust build**, and vice versa.

Consumers: `typecheck:changed` / `test:changed` locally ([inner-loop](inner-loop.md)), the CI job matrix, and the deploy plan below.

## 2. Build vs roll are two different questions

The naive setup filters paths per image, and it is wrong in an expensive way: a one-line edit in a shared package rebuilds four images, and if several services run the *same* image, one digest bump restarts **every pod in the cluster** — several times a day, for code those processes never execute.

Compute **two independent content hashes per unit**:

| Hash | Question | Inputs |
|---|---|---|
| `buildHash` | rebuild the image? | everything the Dockerfile actually bakes in |
| `rollHash` | move this unit's release channel? | only what the unit **executes**, resolved through the workspace dependency graph |

Hash **git blob ids**, not mtimes or file contents read in directory order — then the plan is identical on any machine at the same tree state, and reproducible in CI.

Consequences worth designing for (all deliberate):

- A change under `scripts/` rebuilds the ops image and **restarts nothing**.
- A test-only or root-`docs/`-only PR ships **zero restarts**.
- A change in one service restarts **only** that service.
- A change in the shared DB package restarts **every** service that depends on it — correct, because the pre-sync migration changed the schema for all of them.
- The image in production may be **older than `HEAD`**. The content tag, not the commit, is the identity that matters.

### Under-coupling is the dangerous direction
A missing hash input means a change that **never reaches production** and nothing errors. So don't leave it to review discipline — add a guard that fails CI when a unit's declared hash inputs drift from what its Dockerfile can actually `COPY` → [guards & gotchas](../writing-for-agents/guards-and-gotchas.md).

### The markdown trap
"Exclude `.md` from build hashes" sounds obviously right and silently breaks product content: a docs site *is* markdown, and a blog lives in `content/**`. Exclude **only** prose paths you can name (`docs/` at the repo root, `*.test.ts`), never a global extension. **The worst failure mode of a build-skipping optimization is that nothing errors.**

## 3. One repository per rollable unit

Several processes can run identical bytes (one runtime, an env var picks the entrypoint) and still need **independent rollouts**. Give each rollable unit its **own image name**:

- Build **once**; publish the others as manifest copies (`docker buildx imagetools create`) — seconds, and blobs are shared inside the registry.
- Deploy tooling (kustomize `images:`, ArgoCD Image Updater) keys on **name**, so units sharing one name cannot hold independent digests.

**Tags carry three different meanings** — don't collapse them:

| Tag | Meaning | Moved |
|---|---|---|
| `src-<hash>` | content identity | every build (immutable) |
| `latest` | the channel the deployer watches | only when that unit's own source changed |
| `sha-<commit>` | traceability / rollback | every build |

**Retention protects the digest, not the date.** Keep the newest N by recency *and* whatever digest the channel currently points at *and* anything younger than a rollout window — otherwise a quiet service sitting on a months-old image gets its running image pruned. An age-based rule looks safe and protects the wrong invariant (real dry-run: 236 tags, **0** deletable, so nothing was ever reclaimed). Retention knobs **throw** on a bad value rather than clamping.

## 4. Cache everything that didn't change

| Layer | Cache | Key on |
|---|---|---|
| Dependencies | package manager cache | the lockfile |
| Docker | BuildKit layer cache (`cache-from`/`cache-to`, registry or sticky disk) | Dockerfile + manifest |
| Compiled artifacts | Rust `target/`, TS build info | lockfile + toolchain version |
| Test browsers | the browser download dir | the framework version |
| System deps | a **pre-built base image** | rebuild weekly, not per run |

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/ms-playwright
    key: pw-${{ runner.os }}-${{ hashFiles('bun.lock') }}
- uses: docker/build-push-action@v5
  with:
    cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ matrix.image }}:buildcache
    cache-to:   type=registry,ref=${{ env.REGISTRY }}/${{ matrix.image }}:buildcache,mode=max
```

Also worth the config:
- **Size the runner per job.** A lint job on 2 vCPU, a full test job on 8–16. Paying for 16 vCPU to run a formatter is waste; running the heavy suite on 2 is the queue.
- **`concurrency: cancel-in-progress`** per ref — an agent pushing twice in a minute shouldn't run two full pipelines.
- **A timeout on every job.** A hung job is worse than a failed one: it holds the merge and teaches nothing.
- **Dynamic matrices from the plan.** Emit the matrix as JSON from the detect/plan job so a leg *exists* only for a unit that must be built — a matrix leg that spawns a runner just to gate itself off still costs a minute.
- **Guard the plan job itself.** If the plan can't be computed, fail loudly with a readable error — never let downstream `always()` jobs read a half-written plan and deploy a subset.

## 5. What you must NOT skip

Change detection decides **what to build and restart**, not what to *verify*.

- Never skip lint/typecheck/test for a workspace whose dependency changed — that's exactly the fan-out the detector exists to compute.
- Never skip the whole gate because "only tests changed". A test-only change still has to pass.
- Never let a cache decide correctness. A cache that's wrong must produce a **rebuild**, not a pass.

## Speed targets

| Stage | Target |
|---|---|
| Local scoped check | < 30s |
| Lint (repo) | < 10s |
| Full CI (push → green) | < 5 min |
| Image build (cached) | < 3 min |
| Merge → rolled out | minutes, unattended |

---

**Related:** [inner-loop.md](inner-loop.md) — the local half · [ai-first-cicd.md](ai-first-cicd.md) — the pipeline around it · [linting-ci.md](linting-ci.md) — Biome + runners · [../infrastructure/kubernetes-gitops.md](../infrastructure/kubernetes-gitops.md) — who pulls the image
