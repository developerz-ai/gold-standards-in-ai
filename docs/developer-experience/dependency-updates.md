# 📦 Dependency Updates — two hands: routine waits 14 days, a CVE fix ships now

**A routine update waits 14 days after its release was published; a fix for a known vulnerability merges the day it is known.** The wait protects against an *unknown* risk (a hijacked or broken release, usually caught and pulled by others within days); a published advisory is a *known* risk, and waiting on it only lengthens the exposure. One rule, two hands — never one without the other. As of 2026-11.

## ✋ The two hands

| Hand | Rule | Fails when |
|---|---|---|
| **Routine** — a version update, a new dependency, a lockfile refresh | merges only once every version it introduces was published ≥ 14 days ago | an agent's `bun add` installs a release published that morning |
| **Security** — a version that fixes a published advisory | exempt from the wait, ranked above routine work, merged and deployed at once | a CVE fix sits in the same 14-day queue as a minor bump |

The window is per **release**, read from the registry's publish date — not "14 days after the PR opened". A PR opened late on an old release waits nothing; a rebase onto a newer release restarts nothing but that release's clock.

## 🔒 Where the 14 days hold

| Layer | Shape | Covers | Misses |
|---|---|---|---|
| **Dependency bot** | `cooldown: default-days: 14` on **every** `updates:` entry, every ecosystem | the bot's own version-update PRs | anything an agent or a human adds; an `updates:` entry without the block |
| **Merge gate** | any PR that moves a dependency pin to a release younger than 14 days waits — a lockfile version, an action SHA, an image digest; input = the release's publish date (registry, tag or digest push); a version that fixes an advisory is exempt | **every author** — bot, agent, human | a pin whose publish date it cannot read: start its 14 days from the first time the gate sees the PR, and list that pin kind as unwatched until the gate reads its date |
| **Package manager** | Bun `minimumReleaseAge` (seconds) in `bunfig.toml` | every resolution in the tree | the second hand: it filters the CVE fix too, and its exclude list is a static package list |

**Hold the wait at the merge gate.** It is the only layer that reads both hands at once — the publish date *and* the advisory data — and the only one an agent-authored bump cannot route around. The dependency bot's cooldown is the cheap first layer; a package-manager minimum age is for a repo with no merge gate at all, where it beats nothing.

Dependabot, one entry (repeat the `cooldown:` block on each):

```yaml
version: 2
updates:
  - package-ecosystem: "bun"
    directory: "/"
    schedule:
      interval: "weekly"
    cooldown:
      default-days: 14
    groups:
      bun-minor-and-patch:
        patterns: ["*"]
        update-types: ["minor", "patch"]
```

Dependabot's **security** updates ignore `cooldown:` by design — that is the second hand, already wired. Do not add an `ignore:` that vetoes a security update to keep a pin; fix the pin.

A gate refusal names everything the next debugger needs: package, version, publish date, the date it clears.

## ⚡ The fast lane — the second hand, measured

| Step | Rule | Fails when |
|---|---|---|
| **Watch** | advisory alerts and security updates on for every maintained repo; a scheduled advisory-DB check for what the bot does not read (an MCP server pin, a tool installed by script) | an advisory reaches nobody because nothing subscribed |
| **Rank** | an advisory fix outranks routine work and is exempt from any per-repo work budget | the fix waits behind a refactor |
| **Release** | an advisory-fix commit is a reason to **ship**, never to hold a release | "vulnerability" in a commit subject trips a hardcoded release block |
| **Measure** | record known → filed → merged per fix; report **time-to-merged** and **oldest open advisory age** per repo | "we patch fast" has no number behind it |

## 🚪 Who may merge a dependency PR

| Author | Auto-merge |
|---|---|
| Your dependency bot (Dependabot, Renovate) | yes — after the cooldown and a green gate |
| Your own agent or a member | yes — the same gate, the same 14 days |
| **An outsider** (a fork PR that bumps a dependency) | **never** — a helpful-looking bump from outside is a supply-chain entry. Redo the bump from your own branch if you want it |

## 👀 Ecosystems nothing watches — listed, never silent

Compare the manifests a repo holds with the ecosystems its bot config covers. Each gap is **added** to the config or **written down** as unwatched in the repo's docs — never left silent.

| Usually unwatched | Do |
|---|---|
| A manifest in a subdirectory the config does not list | add the directory to that ecosystem's entry |
| Base images by tag | pin by digest; the `docker` ecosystem moves the digest |
| Actions pinned by tag | pin by SHA; the `github-actions` ecosystem moves it ([workflow security](ai-first-cicd.md#workflow-security--ci-is-a-credential-store)) |
| Container images in workflow `services:` / `docker run` steps | the bot does not read them: bump by hand, list them as unwatched |
| Vendored binaries, `curl \| sh` toolchain installs on boxes | list them as unwatched; give each a scheduled advisory check |

## 📊 Census — prove it, per repo

A scripted, kept census, one row per maintained repo:

| Column | Done when |
|---|---|
| dependency bot present | every repo |
| cooldown days, per `updates:` entry | 14 on every entry |
| ecosystems covered vs manifests found | no silent gap |
| advisory watch on | every repo |
| open advisories · oldest open advisory age | trending to zero · days, not weeks |
| advisory time-to-merged | measured after the first fix merges, then tracked |

---

**Related:** [ai-first-cicd.md](ai-first-cicd.md) — pinned actions and `trustedDependencies` · [../ai-agents/untrusted-input.md](../ai-agents/untrusted-input.md) — config that runs on open · [../writing-for-agents/mcp-json.md](../writing-for-agents/mcp-json.md) — an MCP server pin is a dependency too · [../workflow/shipping-doctrine.md](../workflow/shipping-doctrine.md) — merged green main deploys
