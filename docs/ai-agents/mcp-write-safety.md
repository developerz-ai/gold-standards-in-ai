# 🛡️ MCP Write Safety — split by effect, preview first, `confirmed_by_user` on the irreversible

> As of 2026-10. Proven in production on a multi-tenant finance MCP serving customer agents (Claude and others), and directory-review ready.

An MCP server that writes needs to make three things legible to the agent and the host:
1. **What kind of change each tool makes**: the host decides from that whether to prompt.
2. **What the change will do**: a preview before the commit.
3. **That the user agreed to *this* change**: `confirmed_by_user: true` on the irreversible commit.

Each layer is cheap, and together they're strong. A cooperating model reads all three and follows them. When one is skipped, the audit row shows it.

## ✂️ Split the dispatcher by effect, never by entity

One `manage_resource` that both reads and deletes can't carry an honest `readOnlyHint`, and the Connectors Directory **rejects** it ("separate read and write tools"). Saying which actions are safe in the description doesn't count. Split the tool:

| Tool | Runs | `readOnlyHint` | `destructiveHint` |
|---|---|---|---|
| `list_resources` | catalog + which tool runs each action | `true` | `false` |
| `describe_resource` | one action's full schema | `true` | `false` |
| `read_resource` | every action that changes nothing | `true` | `false` |
| `write_resource` | create / update | `false` | `true`* |
| `delete_resource` | archive / cancel / merge / delete / external send | `false` | `true` |

\* `destructiveHint: false` means "additive only". An update that overwrites a balance or re-files history isn't additive, so mark it destructive.

Rules:
- **Every tool gets a `title`** plus both hints, explicitly. Hosts auto-run read-only tools and always prompt on destructive ones.
- **Classify every action explicitly, with no default.** Reads come from each action's `mutates: false` declaration. Writes sit in two explicit lists (`WRITES`, `DESTRUCTIVE`). An action marked `mutates: true` that's on neither list **fails boot and CI**. A wrong default either hides a delete behind `destructiveHint: false` or makes an ordinary update ask for deletion-grade confirmation.
- **Wrong tool → named refusal.** `delete` sent to `write_resource` returns `{code: "wrong_tool_for_action", use_tool: "delete_resource"}` and never half-runs.
- **Same argument shape on all three** (`resource`, `action`, `params`). The surface stays flat at 5 dispatch tools plus `docs` / `guide`, at any catalog size ([cost table](../writing-for-agents/memory-and-mcp.md#building-mcp-servers--few-parameterized-tools-beat-many)).
- **No money-movement verbs.** The directory doesn't accept connectors that transfer funds. Pin that with a spec that fails on any action named like a payment or transfer.

## 👀 Preview first: `live: false` is the default

Every write that's expensive to undo takes `live`, defaulting to `false`:

```
delete_resource({resource:"customer", action:"merge",
                 params:{canonical_id:"<id>", duplicate_ids:["<id>"]}})
→ {dry_run:true, would_archive:[…], would_move:{invoices:12}}
```

- Run the preview as the real code path inside a **rolled-back transaction**, so preview and commit can't drift.
- Parse booleans **strictly**. If `live: "banana"` raises, the caller never silently previews or silently commits.
- Record `dry_run` on every audit row, derived from the action's *declaration* rather than from the response body.

## ✅ `confirmed_by_user` on the destructive tool

The flag is **required** in `delete_resource`'s `inputSchema`, so the model sees it at connect time in `tools/list`. It doesn't have to wait for a lazy `describe_resource`.

```json
"confirmed_by_user": { "type": "boolean",
  "description": "true when the user approved this specific change. A committing call without true writes nothing and returns confirmation_required with the call it refused. A preview (live false or absent) runs either way." }
```

The flow the model follows: **preview → show the user → user says yes → commit with `confirmed_by_user: true`.**

| Rule | Why |
|---|---|
| Put it on the **tool**, not per action | Required on the tool means it's visible in `tools/list`. A per-action param stays hidden until `describe_resource` |
| Gate only **commits** (`live: true`, or actions with no `live`) | A preview needs no consent, so preview → ask → commit stays two calls |
| Run the gate **after** authz and the permission check, **before** the action body | An unauthorized caller gets `unauthorized`, never a prompt that confirms the action exists. Nothing is written, enqueued or emailed |
| Accept it top-level **or** under `params`, then strip it before param validation | Models misplace it, and an `unknown_params` error would teach the model that the flag is wrong |
| Strict boolean | `"yes please"` is refused, never read as consent |
| Refusal is a fact plus the exact refused call | `{code:"confirmation_required", refused_call:{…}, retry_with:{confirmed_by_user:true}, preview:{…live:false}}`. Everything needed to ask the user, with no prose to parse |
| Record the claim on **every** row (`user_confirmed` column, NULL for other tools) | "How many commits claimed consent" and "how many were refused for missing it" become queries |

### Word it as a fact, not an order
Directory review rejects descriptions that tell the model how to behave. Write *"a committing call without true writes nothing and returns confirmation_required"*, never *"you MUST ask the user first"*. Pin it with a spec regex over every description and every docs body: `/\b(you must|NEVER|ALWAYS|MUST|start here|before calling|if unsure)\b/i`.

## 🧱 The layers together

| Layer | Who enforces it |
|---|---|
| `destructiveHint: true` | The host prompts the user on every call |
| `live: false` default | The server: a forgotten flag previews and never writes |
| `confirmed_by_user: true` | The model reads it and asks the user. The server refuses a commit without it. The audit records the claim |
| Signed preview token (for the riskiest commits) | The server: a commit needs a token the preview issued, bound to the workspace and arguments, with a short TTL |
| Recipient allow-list (for external sends) | The server: it only sends to addresses a human authorised in the app UI |

Use as many as the action's blast radius justifies. Every destructive action gets the first three. Sends that leave the system add the allow-list.

## ✅ Checklist
- [ ] No tool mixes reads and writes. Each one has a `title` plus both hints.
- [ ] Every mutating action is classified explicitly, and an unclassified one fails boot/CI.
- [ ] Expensive writes default to `live: false`. The preview is a rolled-back real run.
- [ ] `delete_resource` requires `confirmed_by_user`. Commits without it are refused with the exact call echoed.
- [ ] Descriptions and docs state facts only, pinned by a regex spec.
- [ ] Audit rows carry `dry_run` and `user_confirmed`.
- [ ] Every tool name is ≤ 64 chars, there are no money-transfer verbs, and every tool has been exercised through the MCP Inspector.

## See also
- [tools-and-mcp.md](tools-and-mcp.md): tool design, the split meta-tool pattern, audited capability access
- [mcp-docs-for-agents.md](mcp-docs-for-agents.md): recipes that show the preview → confirm → commit sequence
- [untrusted-input.md](untrusted-input.md): why the reader/actor split matters when tool output is attacker-controlled
- Connectors Directory review criteria: https://claude.com/docs/connectors/building/review-criteria
