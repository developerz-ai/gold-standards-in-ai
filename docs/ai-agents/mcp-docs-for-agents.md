# 📖 MCP Docs for Agents — teach *their* agent to use *your* server

**Shipping an MCP server isn't shipping an API. It's shipping an API + the onboarding for a junior dev who reads nothing, forgets everything, and starts fresh every session.**

Their agent failing is *your* bug. You don't control their model, their client, or their prompt — you control the text you hand them. Ship the advice *with* the tools.

The cost side of MCP design (few parameterized tools, lazy schemas) is [../writing-for-agents/memory-and-mcp.md](../writing-for-agents/memory-and-mcp.md#building-mcp-servers--few-parameterized-tools-beat-many). This doc is the **comprehension** side.

**Success metric:** *turns-to-first-correct-call, cold.* Replay real transcripts, count wasted calls. Not "endpoints exposed."

## 💬 It's advice, not a script

**You are writing text for a reader, not a program for a runtime.** Nothing on the server executes a recipe — the agent reads it, thinks, and decides. Everything follows from that.

| It is | It is not |
|---|---|
| tips, conventions, warnings, worked examples | a DSL, a state machine, a workflow engine |
| a senior colleague talking over your shoulder | an OpenAPI spec |
| "usually", "prefer", "don't bother", "ask first" | branches the server validates |
| skippable, reorderable, adaptable | a sequence that must run in order |

So write the things a schema structurally *cannot* say:

- **when *not* to do this** — and what to do instead
- **what everyone gets wrong** — the trap, before they fall in
- **local conventions** — "search by email, not name; names aren't unique here"
- **cost and latency** — "this scans everything, filter first"
- **judgment** — "if they already disputed the charge, refunding won't stop the dispute"

Literal call strings are in there because **copying beats composing**, not because it's a program. They're examples inside advice. The prose around them is the point.

## 🚧 Design for the worst client

You can't know who connects. Assume nothing:

| Don't assume | Why |
|---|---|
| `resources/` reach the model | many clients only offer them for a *human* to attach |
| `prompts/` are reachable | user-invoked slash commands in most clients |
| `instructions` (from `initialize`) is injected | Claude Code does it; plenty don't |
| skills exist client-side | you can't install a file on their machine |
| the model plans across turns | it pattern-matches the nearest example |

Guaranteed everywhere, and that's the whole contract:

> **Tool name + tool description reach the model every turn. Tool results come back as text.**

Build inside it. Everything else is an upgrade — set `instructions` too, just never load-bear on it.

## 🍳 The unit is a recipe — a goal, not an entity

A weak model can't compose a plan from a schema. It *can* follow a worked example. Index by **goal**, in the words a user would actually say.

```
docs://recipes/refund-a-payment — refund a payment, full or partial

Use when the customer wants money back and the charge is under 90 days old.
Older than that refunds just fail — issue a credit note instead
(docs://recipes/issue-a-credit-note). Partial refunds are common here:
ask for the amount rather than assuming a full one.

1. manage({resource:"customers", action:"list", filters:{email_eq:"<EMAIL>"}})
   → customer.id        empty? wrong email — stop and ask, don't guess
   skip if you already have customer.id
2. manage({resource:"payments", action:"list", filters:{customer_id_eq:"<customer.id>", status_eq:"succeeded"}})
   → payment.id         created >90d ago? → docs://recipes/issue-a-credit-note
3. manage({resource:"refunds", action:"create",
           params:{payment_id:"<payment.id>", amount_cents:0,
                   reason:"duplicate|fraudulent|requested_by_customer"}})
   → refund.id          amount_cents:0 = full refund; else partial, in cents

Done when refund.status == "pending".

Tips
- Search customers by email, not name — names aren't unique here.
- A payment can carry several refunds. Sum the existing ones before a partial,
  or step 3 fails with REFUND_EXCEEDS_PAYMENT.
- "pending" lasts 5–10 days. That is normal. Never retry a pending refund.
- Already disputed? Refunding does not stop the dispute. Check payment.dispute_id.
- Refunds are irreversible. If the amount is unclear, ask — don't pick one.
```

Why each part earns its place:

| Element | Why a weak model needs it |
|---|---|
| "Use when…" opener | lets it bail early into the right recipe |
| literal call string | zero composition — copy, substitute, send |
| `<customer.id>` placeholder | the name matches the previous step's `→`; substitution is lookup, not inference |
| `→ gives` | data flow made explicit instead of implied by prose |
| `skip if` | lets it enter mid-recipe when it already holds an id |
| one branch per step | the common failure, *before* it fires the call |
| `Done when` | it knows to stop instead of retrying |
| Tips | the judgment a schema can't carry |

Never `$1`, `{{var}}`, or "call the list action filtering by email" — each needs a reasoning step you're trying to avoid.

### Self-sufficient in one read
It reads once, then executes. It will not come back mid-sequence.

- **Inline enums and required fields** on the step that needs them. Sending it to `describe_resource` mid-recipe burns a turn and invites derailment.
- **Warnings go on the step they bite**, never only in a trailing section — by then it has already fired.
- **One branch per step, not a decision tree.** The tree is where a weak model gets lost.
- **Never chunk a recipe.** Half a recipe is worse than none. Doesn't fit the cap → it's two recipes.

### Plain text, not JSON
Same recipe as JSON costs ~⅓–½ more tokens, and size isn't the real win:

- **No escaped quotes.** `\"customers\"` gets copied into the real call verbatim, and it fails.
- **`→` is one token**; `"gives": "customer.id"` is six-ish.
- **Keys stop repeating** once position makes them obvious.
- **Prose has nowhere to live in JSON.** The tips are the point; don't shove them in a `"note"` field.

Keep the *shape* (uri, "use when", numbered steps, `→`, skip-if, branch, done-when, tips). Drop the braces. Exception: keep JSON where the client consumes it programmatically (`outputSchema`) — and never prettify the call strings themselves.

## 🗂️ Shape it like `resources/` — list → read

URI-addressed, delivered through a **tool** so every client gets it.

```json
{
  "name": "docs",
  "description": "How to use this server. Call docs({}) first if unsure how to sequence calls.\nReturns short guides with literal, copy-pasteable calls plus the gotchas.\n  docs({})                              -> list all docs\n  docs({q:\"refund a payment\"})          -> search\n  docs({uri:\"docs://recipes/refund\"})   -> read one\nCommon: refund a payment - issue a credit note - find a failed webhook - cancel a subscription",
  "inputSchema": {
    "type": "object",
    "properties": {
      "uri": { "type": "string", "description": "docs:// URI from a listing" },
      "q":   { "type": "string", "description": "plain-English goal, e.g. 'give money back'" }
    }
  }
}
```

No `required` — **`docs({})` must work**. Weak models call tools with `{}`; make that useful instead of a validation error.

`docs({})` → the listing:
```
ACME docs — call docs({uri:"..."}) to read one.

RECIPES (do this)
  docs://recipes/refund-a-payment        refund fully or partially
  docs://recipes/issue-a-credit-note     credit when the refund window expired
  docs://recipes/find-a-failed-webhook   why a webhook never arrived
  docs://recipes/cancel-a-subscription   cancel now or at period end

REFERENCE (look this up)
  docs://ref/filters                     filter operators: _eq _in _gt _cont
  docs://ref/errors                      error codes -> which recipe fixes it
  docs://ref/conventions                 ids, money, timezones, pagination

RESOURCES (act on these)
  customers, payments, refunds, credit_notes, subscriptions, webhooks
  → manage({resource:"payments", action:"list", filters:{...}})
  → describe_resource({resource:"payments"}) for fields and enums
```

`docs({q:"give the customer their money back"})` → **inline the top hit**; search-then-read costs a turn for nothing:
```
Best match — docs://recipes/refund-a-payment

<the recipe body, inlined>

Also: docs://recipes/issue-a-credit-note · docs://ref/errors
```

A miss **never returns empty** — zero results makes a weak model conclude the capability doesn't exist and improvise:
```
No doc at that URI. Closest:
  docs://recipes/refund-a-payment        refund fully or partially
  docs://recipes/issue-a-credit-note     credit when refund window expired
Full list: docs({})
```

**Errors point back at the docs** — the highest-value link in the system; it arrives exactly when the agent is stuck, and costs it no search:
```json
{ "error": "REFUND_WINDOW_EXPIRED",
  "message": "payment is 142 days old; refunds are limited to 90 days",
  "docs": "docs://recipes/issue-a-credit-note" }
```

**Mirror the same URIs as real MCP resources** — one store, two surfaces. Smart clients get it natively; the tool covers everyone else.
```
resources/list  → [{uri:"docs://recipes/refund-a-payment", name:"Refund a payment", mimeType:"text/plain"}, …]
resources/read  → { contents:[{uri, mimeType:"text/plain", text:"<same body>"}] }
```

Never let the resource copy be the only copy.

## 📏 The surface stays constant

| Tool | Answers |
|---|---|
| `docs` | *how do I…* → recipes + the map |
| `list_resources` | *what's in here* |
| `describe_resource` | *what fields/enums does X take* |
| `manage_resource` | do it |

Four tools, ~800 tokens, flat at any catalog size.

- **One `docs` tool, not `docs_search` + `docs_read`.** Two names = two chances to pick wrong; the server knows whether it got `q` or `uri`.
- **Keep `docs` separate from `describe_resource`.** Different question shapes. A recipe crosses three resources — folding it into `describe_resource` makes it findable only if the model already guessed the right resource, which is the guess it can't make.
- **Cross-reference in the other descriptions:** end each with `If unsure how to sequence calls, start with docs({q:"..."})`. ~12 tokens, in context every turn, catches the model before it improvises.
- **Keep the description under ~1k chars.** Some hosts truncate long ones with no error — the tail just vanishes. Entry-point call on the *first* line.
- **Return flat plain text.** No markdown tables, no nested JSON. Clients strip formatting, truncate, or hand over raw text; plain lines survive all of it.

## 🔎 Skip search until you need it

A title line is ~12 tokens.

| Recipes | Do this | Cost |
|---|---|---|
| ≤30 | whole list in the tool description + `instructions` | ~350 tok, always in context, **zero calls** |
| 30–200 | `docs({})` returns the full list, grouped | one turn, once per session |
| 200+ | `docs({q})` with ranking | one turn per lookup |

Most servers never leave the first row. **Search is what you avoid needing, not a feature.**

When you do need it:
- **Ship `aliases` per recipe** — `refund, money back, reverse a charge, chargeback`. With aliases, plain lexical/BM25 is enough; no embeddings, no index to keep warm.
- **Return 1–2 whole recipes**, ranked hard. A recipe is paid once but sits in the transcript all session — retrieval *precision* beats trimming.
- **`docs({})` is the fallback for everything**: bad uri, bad query, no args.

## ✏️ Easy to edit or it rots

**The maintainer must be able to fix a doc in 30 seconds, without touching code.** Any friction — a schema to learn, a build step, a redeploy — and the docs drift from the API within a month. A stale recipe is worse than none: it burns turns *and* teaches confident wrong behavior at scale.

So: **one plain markdown file per doc, served verbatim.**

```
server/
  docs/
    recipes/refund-a-payment.md
    recipes/issue-a-credit-note.md
    ref/filters.md
    ref/errors.md
```

```markdown
---
uri: docs://recipes/refund-a-payment
title: refund a payment, full or partial
aliases: [refund, money back, reverse a charge, chargeback]
---

Use when the customer wants money back and the charge is under 90 days old.
Older than that refunds just fail — issue a credit note instead
(docs://recipes/issue-a-credit-note).

1. `manage({resource:"customers", action:"list", filters:{email_eq:"<EMAIL>"}})`
   → customer.id — empty? wrong email, stop and ask
...

Tips
- Search customers by email, not name — names aren't unique here.
- "pending" lasts 5–10 days. Never retry a pending refund.
```

Three frontmatter fields, and the body is whatever the maintainer wants to say.

| Property | Why it matters |
|---|---|
| **Body served byte-for-byte** | what you edit is exactly what the agent sees — no rendering surprises |
| **No schema for the prose** | a tip that doesn't fit a field never gets dropped |
| **Listing generated from frontmatter** | drop in a file, it appears in `docs({})` — no registry to update |
| **Hot-reloaded from disk** | fix a typo without a deploy; watch the dir, rebuild the index |
| **Plain diffs** | a docs fix is a normal PR, reviewable by anyone, not a code change |
| **An agent can maintain them** | point Claude at `docs/` — it's markdown, the thing it's best at |

Frontmatter earns its keep only for what the *listing* needs. Resist adding fields.

### Keep it honest
- **Fence the calls, replay them in CI.** Extract every `` ` ``-fenced `manage({...})` line and run it against seeded fixtures. Rename a param → the test fails in *your* repo, not silently in a customer's agent. Same gate discipline as [../developer-experience/ai-first-cicd.md](../developer-experience/ai-first-cicd.md).
- **Lint the links.** Every `docs://` URI in a body must resolve to a file. Dangling URIs send agents in circles.
- **Live next to the handlers**, in the server repo. Not a wiki, not a CMS, not a database.
- **One recipe per real intent, not per endpoint.** Derive them from transcripts and support tickets; ~20 covers most traffic.
- **Every support ticket is a missing tip.** When a human explains something twice, it belongs in a file.
- **Name them `recipes/<verb>-a-<noun>`** so a model that half-remembers a URI can still construct one.

## 🧩 Skill or tool? It's a distribution question

Same architecture — a skill *is* progressive disclosure (description in the system prompt, body on demand). A recipe is basically a skill body. What differs is **who ships it**.

| | Skill | `docs` tool |
|---|---|---|
| Lives | client side (repo, plugin, `~/.claude/skills`) | server side |
| Updates | whoever owns that machine — **drifts** | you deploy, every client is current |
| Reaches | Claude-family clients | any MCP client |
| Cost | ~30 tok description | ~200 tok tool def |
| Carries scripts/files | yes | text only |
| Round-trip | none — it's a file read | one call |

- **Your own agents + your own server** → **skills.** Cheaper and simpler; skip the fourth tool. See [../writing-for-agents/skills-commands-agents.md](../writing-for-agents/skills-commands-agents.md).
- **Clients you don't control** (customers, third-party agents, non-Claude models) → **the tool.** The only channel you own, and the only one that stays in sync with the deployed API.
- **Both** → one folder of markdown, CI-tested; generate the skill files *and* serve them from `docs`.

**The combination worth stealing:** the skill carries *strategy* (when to use this server, the 5 daily recipes, house conventions, what not to do — stable), the tool carries *current truth* (full list, exact params, whatever shipped this morning). The skill ends with "for anything not listed here, call `docs({q:...})`."

*As of 2026-09: skills-served-over-MCP is moving fast — check current client support before betting on it; it would collapse this tradeoff entirely.*

## ✅ Checklist

- [ ] `docs({})` works with no args and returns the map
- [ ] Tool description ≤1k chars, entry-point call on line 1, 3–5 common recipe titles listed
- [ ] Same titles in `initialize.instructions` (upgrade, not a dependency)
- [ ] Every recipe opens with "use when" and names the alternative
- [ ] Literal calls, `→ gives`, inline enums, one branch per step, `Done when`
- [ ] Tips section carries the traps, conventions, costs and irreversibles
- [ ] Search inlines the top hit; a miss returns nearest + `docs({})`
- [ ] Every error carries a `docs://` URI
- [ ] Same URIs mirrored on `resources/list` + `resources/read`
- [ ] Other tools' descriptions end with "start with `docs({q:...})`"
- [ ] One markdown file per doc, body served verbatim, hot-reloaded
- [ ] CI replays every fenced call; link-lint every `docs://` URI

## See also
- [tools-and-mcp.md](tools-and-mcp.md) — tool design, the 3-tool meta pattern, audited capability access
- [../writing-for-agents/memory-and-mcp.md](../writing-for-agents/memory-and-mcp.md) — the token-cost case for few parameterized tools
- [context-budget.md](context-budget.md) — lazy surfaces, chunked results, prompt-cache prefix order
- [../writing-for-agents/compressed-config.md](../writing-for-agents/compressed-config.md) — how to write the text you return
