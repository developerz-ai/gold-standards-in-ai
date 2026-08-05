# 🤖 AI-First CI/CD

When agents open most of the PRs, CI stops being a safety net for humans and becomes the **agent's feedback channel**. Design it for that reader: one command it can run locally, identical in CI, failures that name the fix, and a deploy it can verify without asking anyone.

## 1. One gate, two places — and a test that proves it

The agent must be able to earn certainty *before* pushing. That only works if the local gate and CI run the **same commands**.

```
bin/check  ≡  the CI job
   ├── stores preflight        (integration deps up)
   ├── typecheck  (all workspaces)
   ├── lint       (formatter + every custom guard)
   ├── test       (unit, then integration)
   └── compiled languages (cargo fmt --check, clippy -D warnings, test --all-features)
```

**Green locally ⇒ green CI.** Anything else teaches the agent that pushing is how you find out — which is the slowest possible loop.

### The parity test
Drift is invisible until it costs a day, so **assert it**: a test that parses the CI workflow and the local gate script and fails when they diverge.

```ts
// scripts/lint/ci-lint-parity.test.ts
it("CI runs every check bin/check runs", () => {
  expect(ciSteps()).toEqual(expect.arrayContaining(localGateSteps()));
});
```

Same shape for any generated artifact: commit it, and add a `--check` mode that regenerates and diffs.

```bash
bun scripts/docs/gen-tool-inventory.ts --check   # fails lint if the committed doc is stale
```

Real failure this prevents: a check that exists only in CI (`lint` was repo-wide there, scoped locally) makes agents push red repeatedly and blame the flake.

## 2. Only do the work that changed

Path-filter the pipeline and reuse the **same** affected-detection logic as the local `:changed` commands ([inner-loop](inner-loop.md)). A TS-only PR must never compile Rust; an unchanged app must never rebuild an image.

```yaml
jobs:
  detect:
    outputs: { api: ${{ steps.f.outputs.api }}, router: ${{ steps.f.outputs.router }} }
    steps:
      - id: f
        uses: dorny/paths-filter@v3
        with:
          filters: |
            api:    ['apps/api/**', 'packages/**']
            router: ['apps/router/**']
  api:    { needs: detect, if: needs.detect.outputs.api == 'true',    uses: ./.github/workflows/ts.yml }
  router: { needs: detect, if: needs.detect.outputs.router == 'true', uses: ./.github/workflows/rust.yml }
```

Cache everything else ([linting-ci](linting-ci.md#cicd-is-the-bottleneck--cache-everything)). Target: **push → green in < 5 min**, because that number multiplies by every PR an agent opens.

## 3. Gates that aren't tests

An AI-first repo enforces things a unit test can't see. Each is a CI job, each fails loudly:

| Gate | Guards against | Shape |
|---|---|---|
| **Custom lint guards** | repo-specific defect shapes recurring | `scripts/lint/*.ts`, one rule per file → [guards-and-gotchas](../writing-for-agents/guards-and-gotchas.md) |
| **Budget tests** | prompts/tool surfaces/skill catalogs silently bloating | assert byte or token ceilings on what ships in the standing context → [context-budget](../ai-agents/context-budget.md) |
| **Generated-doc `--check`** | a committed inventory going stale | regenerate + diff |
| **Dialect-compat guard** | a migration your dev engine accepts and prod's doesn't | static check on migration SQL → [data-and-scale](../architecture/data-and-scale.md) |
| **Secret scan** | a key in a diff | a PR job; pin third-party actions to a SHA |
| **Agent evals** | a "cheap" prompt change breaking tool routing | **not** in the default gate — see below |

### Evals are a pre-merge gate, never a CI job
Agent-behavior tests (does the model still pick the right tool? still load the right skill?) call **live models**: they cost money and they're stochastic. Keep them out of `bun test` globs and out of CI; run them deliberately before merging a change to prompts, tools, or skills, and paste the pass rates into the PR. A flaky paid job in the default gate gets muted within a week — and a muted gate is worse than none.

## 4. Never push red — and never merge blind

- **Never push red.** The gate exists so the agent doesn't discover breakage in front of a reviewer.
- **A PR with 0 checks registered is not a passing PR.** Auto-merge right after a rebase can land red because checks haven't attached yet — wait for them to register.
- **Cap PR size at ~110–120 changed files.** Automated reviewers skip past a limit (CodeRabbit: >150 → "Review skipped"), so the riskiest PR gets the least review → [workflow-commands](../writing-for-agents/workflow-commands.md#-the-seam--one-continuous-line).
- **AI review on every PR** (CodeRabbit or equivalent) with path-scoped instructions, then a human merges → [linting-ci](linting-ci.md#automated-ai-code-review).

## 5. Failures must be readable by the agent that caused them

CI output *is* a prompt. Optimize it:

- **Fail fast, one cause per job** — don't bury the real error under 400 lines of a second failure.
- **Name the fix in the message**: `apps/api/src/routes/x.ts:42 threw a bare Error — use an AppError subclass (src/errors/). scripts/lint/no-bare-error.ts`.
- **Expose logs to the agent**: `gh run view --log-failed` in the allow list, and a `/fix-ci` command that pulls the failing job, reproduces locally with the *scoped* command, fixes, pushes.
- **Flaky ≠ acceptable.** A flaky job trains the agent to re-run instead of read. Quarantine it with an issue, not a `continue-on-error`.

## 6. Merge → deploy without a human bottleneck

GitOps handoff: CI builds and pushes images, the cluster pulls them. **CI holds zero cluster credentials** → [kubernetes-gitops](../infrastructure/kubernetes-gitops.md).

- **Migrations self-deploy** (a pre-sync job). Treat that as a design constraint, not a convenience: a migration merges ⇒ it runs, everywhere, once, forever → [data-and-scale](../architecture/data-and-scale.md#-migrations-vs-backfills).
- **Data rewrites run themselves too** if you wire a post-sync backfill job — so write one as if it executes the moment it merges, because it does.
- **Content-hash the build**: rebuild only units whose inputs changed; roll the rest.
- **Verify the roll, don't assume it.** Confirm the deployed build id from a live probe (`/version`, a build-sha meta tag, pod image digest) — *never* by grepping the bundle for a string; code-splitting will lie to you.
- **Env vars are a contract**: a new runtime variable ships with its `.env.example` entry and an explicit line in the PR description for whoever mirrors it into the cluster. A ConfigMap change usually needs a rollout restart to take effect — say that in the runbook.

## 7. Production errors are the pipeline's last stage

Wire the error monitor into the loop: a new issue → an agent reads the trace, writes a reproducing test, opens the PR. The deploy isn't "done" when the roll finishes; it's done when the symptom is gone → [observability](../infrastructure/observability.md).

---

**Related:** [inner-loop.md](inner-loop.md) · [linting-ci.md](linting-ci.md) · [guards-and-gotchas.md](../writing-for-agents/guards-and-gotchas.md) · [../workflow/shipping-doctrine.md](../workflow/shipping-doctrine.md)
