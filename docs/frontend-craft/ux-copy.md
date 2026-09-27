# ✍️ UX Copy, Onboarding & Information Clarity

Words are interface. Every string answers: **what happened, what matters, what to do next.** Voice comes from the product, not from the model's default register. Two apps built with these standards must not *sound* alike any more than they look alike.

**Scope.** Voice, microcopy, states, forms, onboarding, cutting, page shaping. Related:

| Need | Go to |
|---|---|
| Short copy ban list, visual floor | [design-taste.md](design-taste.md) |
| Harden checklist (empty/long/missing data) | [design-review-loop.md → Harden](design-review-loop.md#-harden-design-for-real-data-not-demo-data) |
| Pick the visual world the words must match | [design-directions.md](design-directions.md) · [brand-identity.md](brand-identity.md) |
| Success/celebration moments, motion tone | [motion-and-delight.md](motion-and-delight.md) |
| Copy that adapts per device/density | [adaptive-ui.md](adaptive-ui.md) |
| Copying text from a reference image | [reference-image-design.md](reference-image-design.md) |
| Catalog keys, plurals, locales | [i18n.md](i18n.md) · [dates-money-timezones.md](dates-money-timezones.md) |
| Record voice in the project | [design-md.md](design-md.md) |

## 🧭 Order of operations
1. **Shape** the surface from the job-to-be-done (below). No copy before the job is named.
2. **Derive voice** from product truth → write 3 voice words + 3 anti-words into `PRODUCT.md` / `DESIGN.md`.
3. **Draft real content** at real ranges (min / typical / max). Never lorem.
4. **Write by function:** actions, states, forms, errors, onboarding.
5. **Distill:** cut words and UI until it breaks, then put back one thing.
6. **Audit** every visible string (self-audit below) → [review loop](design-review-loop.md).

## 🗺️ Shape the page from the job
Page structure comes from the visitor's job, not from "landing pages usually have…".

Answer before layout (max 2–3 questions per round to the user; assert the likely reading, invite correction):

| Question | Output |
|---|---|
| Who arrives, from where, in what state of mind? | Visitor mode: scanning, stressed, curious, returning expert |
| The ONE thing they must understand or do? | Primary action. Exactly one per surface |
| What is uniquely true here? | The proof a neighboring product can't claim |
| What real content must it carry? Min/typical/max? | Content ranges for layout + truncation |
| Which states matter? | First-run, empty, loading, error, success, no-permission, overflow, expert |
| What must not change? What would feel wrong even if polished? | Anti-goals |

**Message hierarchy per state** (decide in this order):
1. The one fact the user needs now.
2. The action available next.
3. Context that *changes the decision* (nothing else).
4. Tone for this moment.

- **Content-first IA.** Order sections by the questions the visitor asks, in the order they ask them. Name nav items with the user's nouns, not the org chart (`Invoices`, not `Billing Module`).
- **One primary action per surface**, few secondary, rest tertiary or hidden.
- **Progressive disclosure** for uncommon detail: accordion, "Advanced", step-through. Never hide what a decision needs.
- **Brief = 3–5 bullets** when settled: job + audience, outcome + proof, scope + anti-goals, states + ranges, open decisions the builder must not invent.

## 🎙️ Voice & tone per app
**Voice is constant per product. Tone shifts per moment.** Voice is derived, never defaulted. The model's default voice (upbeat, hedged, vaguely inspirational) is the copy equivalent of Inter + purple gradient.

### Derive voice
1. One sentence of product truth: what, for whom, in what scene.
2. **Three voice words + three anti-words.** Anti-words name the rut to avoid.
3. One **sample line** per state (button, empty, error, success) written in that voice.
4. Record in `PRODUCT.md` (truth, audience, voice) + `DESIGN.md` (sample lines). Check sibling apps; differ on ≥1 voice word.

| Product | Voice words | Anti-words | Empty-state line |
|---|---|---|---|
| Incident pager for SREs | terse, exact, calm | cute, chatty, alarmist | `No open incidents.` |
| Kids' reading app | warm, playful, short | sarcastic, adult, preachy | `Pick a book to start your shelf.` |
| Tax filing tool | plain, reassuring, precise | jokey, legalese, salesy | `No documents yet. Upload last year's return to prefill 30+ fields.` |
| Design critique tool | expert, decisive, editorial | hedging, hype, teacherly | `Nothing to critique yet. Point it at a URL.` |
| Indie game store | loud, fan-voiced, irreverent | corporate, neutral, bland | `Your library's empty. Go find a weird one.` |

### Tone by moment
| Moment | Tone | Rule |
|---|---|---|
| Routine success | Quiet | Past-tense fact. `Saved.` No `!` |
| Milestone (first publish, first payout) | Warmer, may expand | Name the outcome + next step. Once, not every time |
| Waiting | Honest | Name the real operation. Never fake progress |
| Error, blocked work | Serious, calm | Problem + recovery first. Warmth OK, jokes never |
| Money, privacy, deletion, access loss | Plain, exact | Zero humor. Name the consequence |
| First use | Inviting, brief | Next action before personality |
| 100th use | Invisible | Must still be pleasant on repeat. No rotating quips |

- **Generic whimsy < neutral clarity.** Personality lives in the product's own nouns, not in puns.
- **One register per page.** Don't mix terminal-speak, editorial prose, marketing punch.
- **No hedging** when the product knows: `Deletes 12 files` not `This may delete some files`.
- **Hedge honestly** when it doesn't: `Usually takes under a minute` beats a fake countdown.
- **Glossary.** Same concept = same word everywhere (`workspace` never also `team`/`org`/`space`). Keep a short term list in `DESIGN.md`. Never vary words for literary effect.

## 🚫 AI-slop copy (deeper list)
Base list lives in [design-taste.md](design-taste.md). The patterns below are what survives after banning the obvious words.

| Pattern | Example tell | Fix |
|---|---|---|
| Verb-of-transformation | `Transform your workflow`, `Reimagine X`, `Take X to the next level` | Say the concrete change: `Close the books in 2 days, not 8` (real number) |
| Empty intensifier | `Effortlessly`, `truly`, `powerful`, `robust`, `cutting-edge`, `world-class` | Delete. Show the capability instead |
| Aphorism triplet | `Fast. Simple. Yours.` / `X. No Y.` repeated | One specific sentence |
| Mock-poetic micro-meta | `Built with care, not hype.` / `The quiet part, said loud.` | Delete or replace with a fact |
| Fake-craftsman labels | `Hand-tuned`, `Crafted`, `Artisanal pipeline` | Delete unless literally true |
| Pseudo-enterprise jargon | `Orchestration layer`, `runtime markers`, `00 · SYSTEM` | Plain function label or nothing |
| Passive-aggressive humility | `We're just getting started.` / `No promises, but…` | Cut |
| Unclear referent | `We plan to stay that way.` (which way?) | Rewrite self-contained |
| Motivational filler | `Unlock your potential`, `Smarter than ever`, `Your journey starts here` | Name the task |
| Rhetorical question hero | `Tired of spreadsheets?` | State the outcome |
| Chat-assistant voice in UI | `Great question!`, `Sure! Here's…`, `I hope this helps` | Remove; UI is not a chat reply |
| Em/en dash as separator | `Fast — and free` | Period, comma, colon |
| Exclamation on success | `Done!` / `Awesome!` | `Done.` |
| `Oops!` / `Uh oh!` / `Whoops` | `Oops! Something went wrong` | What failed + fix |

## 🧑‍💼 Realistic content, not placeholders
**Content is design material.** Layout tested on lorem lies about wrapping, rhythm and density.

- **Author at production fidelity:** names, entries, titles, prices, dates, avatars. Every blank the brief left open is yours to write.
- **Names:** locale-appropriate, diverse, unique per row. Not John Doe / Jane Smith / Test User.
- **Brands:** believable and in-world. Not Acme / Nexus / NovaCore / Flowbit / Quantix.
- **Numbers:** organic (`$1,284.50`, `47.2%`), not round (`$100.00`, `50%`) or fake-precise vanity (`99.99%`, `10x`).
- **Dates:** varied, plausible, relative where the product uses relative time.
- **Ranges:** include one very long value (42-char name, 3-line title) and one very short.
- **Claims are never invented:** customers, testimonials, benchmarks, prices, certifications. Label demo data visibly (`Sample data`) or leave a `<!-- TODO: real testimonial -->` slot and list it for the user.

## 📢 Headlines & CTAs
| Element | Rule |
|---|---|
| Hero headline | ≤ 8 words ideal, ≤ 2–3 lines. Outcome or mechanism, not category (`Invoices that chase themselves`, not `Modern invoicing platform`) |
| Subhead | ≤ 20–25 words. Adds *how* or *for whom*. Never restates the headline |
| Section heading | ≤ 8 words. The section's answer, not its topic (`Pay in 38 currencies` > `Global payments`) |
| Primary CTA | Verb + object, ≤ 3 words, one line. Describes the outcome, not the gesture (`Start free trial`, not `Click here`/`Submit`) |
| Secondary CTA | Max 1 next to primary. Different intent (`See pricing` next to `Start free trial`) |
| CTA intent | One label per intent per page. Pick `Book a demo` and reuse it verbatim |
| Link text | Makes sense out of context. Not `here`, `learn more` alone → `Read the refund policy` |
| Casing | Sentence case everywhere. Same casing for the same element type |

- **Memory test:** after one viewport, what would the visitor repeat an hour later? If the answer is a mood, rewrite.
- **If the value prop won't fit in two lines, the idea isn't sharp yet.** Cut words, don't shrink type first.

## 🔘 Microcopy by function
| Surface | Rule | ❌ Before | ✅ After |
|---|---|---|---|
| Button | Verb + object; outcome not gesture | `Submit` | `Send invoice` |
| Button | Same verb as the heading/dialog | Heading `Archive project` / button `OK` | `Archive project` |
| Toggle/checkbox | State the on-condition | `Notifications` | `Email me when a build fails` |
| Loading | Name the operation; honest time | `Loading…` | `Importing 1,240 contacts… about 30 seconds` |
| Success | Past tense, brief, no `!` | `Success! Your changes have been saved!` | `Changes saved.` |
| Success w/ consequence | Mention only if it changes what to do next | `Invite sent!` | `Invite sent. Maya can join until Friday.` |
| Tooltip | Answers the implicit question, not the label | `Sync: syncs data` | `Pulls new orders every 15 minutes` |
| Icon-only control | Accessible name = the visible outcome | `aria-label="icon"` | `aria-label="Delete draft"` |
| Badge/status | User's words, not system state | `STATE_PENDING_REVIEW` | `Waiting for review` |
| Timestamp | Relative near, absolute far | `2026-09-27T14:03:11Z` | `3 min ago` / `12 Mar 2025` |

## ❗ Errors
**Every error answers:** what failed · why (only if known and useful) · how to recover or what alternative remains.

| ❌ Before | ✅ After |
|---|---|
| `Oops! Something went wrong.` | `Couldn't save your draft. Check your connection, then try again.` |
| `Error 422: Unprocessable Entity` | `That date is in the past. Pick a date from today on.` |
| `Invalid input` | `Phone number needs a country code, like +44 20 7946 0958.` |
| `You entered the wrong password.` | `Email or password doesn't match. Reset password?` |
| `Payment failed.` | `Your card was declined. Try another card, or contact your bank.` |
| `Access denied.` | `Only workspace admins can change billing. Ask Priya Nair to update it.` |
| `Upload error` | `photo.heic is 42 MB. Max is 20 MB. Export as JPG or compress it.` |

- **No blame.** Subject = the system or the thing, not `you` (`That code expired` > `You entered an expired code`).
- **No raw codes as headline.** Codes go in small print for support.
- **No promised cause** the system can't know. Never guess.
- **Keep user input** on error. Never clear a form.
- **Inline over modal.** Never `window.alert()`.
- **Announce** errors to screen readers (`role="alert"` / `aria-live`), and never by color alone.

## 🗑️ Confirmations & destructive actions
**Prefer undo over confirm** when recovery is safe. Confirm only when the action is irreversible, expensive, or affects others.

| Level | Pattern | Copy |
|---|---|---|
| Reversible | Act now + undo toast | `Moved 3 files to Trash. Undo` |
| Irreversible, one object | Dialog naming object + consequence | Title `Delete "Q3 forecast"?` · Body `This removes it for all 6 collaborators. You can't undo this.` · Buttons `Delete forecast` / `Keep it` |
| Irreversible, high blast radius | Type-to-confirm | `Type the workspace name to delete it and its 214 projects.` |
| Affects others | Name who | `Remove Tomás from Design? He loses access to 12 files.` |

- **Never `Yes` / `No` / `OK` / `Cancel`** as the only labels. Button repeats the verb + object.
- **Cancel option = safe default**, focused first, labeled with the outcome (`Keep it`, `Go back`).
- **Numbers make it real:** state counts and names, not `some items`.

## 📝 Forms
| Rule | Detail |
|---|---|
| Persistent label | Above the input. Placeholder = example, never the label (it vanishes on typing) |
| Requirements before, not after | Format, length, eligibility shown before submit (`8+ characters, one number`) |
| Required vs optional | Mark the minority consistently. Mostly required → mark `(optional)` |
| Why we ask | Only when not obvious: `Phone, for delivery updates only` |
| Validate | On blur or submit, not on every keystroke. Error below the field, next to the problem |
| Error wording | What + how to fix, no blame: `Enter a date after 1 Jan 2020` |
| Ask less | Every field must earn its place. Collect later what isn't needed now |
| Smart defaults | Pre-fill country, currency, timezone from locale. Only ask when needed |
| Submit label | The outcome: `Create account`, `Pay $48.00`, not `Submit` |
| Field names | User words: `Company name`, not `org_display_name` |

## 🫙 Empty & zero states
Empty is never blank. **Say which empty it is**, then give the next action.

| Case | Emphasis | Example |
|---|---|---|
| First use | Value + first action + template | `No projects yet. Projects keep files, tasks and people together.` [`Create project`] [`Start from template`] |
| User cleared everything | Light touch, easy recreate | `Inbox zero.` |
| No search results | Echo query, suggest fix | `No results for "invocie". Did you mean "invoice"?` [`Clear filters`] |
| Filtered to nothing | Name the filter | `No paid invoices in March.` [`Show all months`] |
| No permission | Why + how to get access | `You need editor access to see drafts.` [`Request access`] |
| Failed to load | What happened + retry | `Couldn't load invoices.` [`Try again`] |
| Not yet (async) | When it will appear | `Reports appear after your first full day of sales.` |

Visual: one icon or illustration from the product world, not a generic empty-box. Contextual help link only if a real doc exists.

## 🚀 Onboarding
**Job: get to the activation moment, not teach the product.** Activation = the first action that proves value (first invoice sent, first deploy green, first friend added). Name it before designing anything.

| Principle | Rule |
|---|---|
| Time to value | Front-load the 20% that gives 80%. Everything else: contextual discovery |
| Show, don't tell | Real functionality, not a separate tutorial mode |
| Optional | Visible `Skip`. Never block the product |
| Context over ceremony | Teach a feature when the user first meets it. Empty states *are* onboarding |
| Respect intelligence | No explaining standard patterns |
| Once | Track seen/dismissed per user server-side. Never show twice. Never re-onboard returning users |

### First-run sequence
1. **Welcome** (optional): what this is + honest time estimate (`Takes about 2 minutes`) + `Skip`.
2. **Setup:** minimum fields. Explain why for each non-obvious ask. Defaults everywhere.
3. **1–3 core concepts max**, interactive, with progress (`Step 2 of 3`).
4. **First success:** user does something real, pre-filled with a template or sample data.
5. **Acknowledge** proportionally (one line, not confetti for a profile photo) + **one** next step.

### Pick the pattern
| Pattern | Use when | Rules |
|---|---|---|
| Empty-state onboarding | Default for most apps | Value line + first action + template |
| Checklist | Setup has 3–5 independent steps with real payoff | Each item = an outcome (`Connect your bank`), show progress, dismissable, disappears when done |
| Contextual tooltip | One feature, first encounter | Points at the element, 1 sentence + benefit, `Don't show again` |
| Guided tour | Dense expert UI, or big redesign | 3–7 steps, workflow not features (`Create a project` not `This is the project button`), user clicks real buttons, skippable, replayable from Help |
| Sandbox + sample data | High stakes or unfamiliar concepts | Clearly labeled `Sample data`, one-click clear, clear objective, graduation moment |
| Feature announcement | Shipped something users should try | What + why it matters + `Try it`, dismissable, once |

- **Sample data beats tours.** A populated dashboard teaches layout faster than seven spotlight bubbles. Label it and make removal one click.
- **Checklists over tours** when steps are independent. Tours over checklists when order matters.
- **Never:** long forced flow before use · hidden `Skip` · tooltips that reappear · dimming the whole UI with no way out · tutorial disconnected from the real product.
- **Measure:** time to activation, step drop-off, skip rate (high = too long or no value).

## ✂️ Distill: cut until it breaks
Simplicity = removing obstacles between user and goal, not removing features.

1. Name the **one** job of this surface.
2. For every element and sentence: does the job need it? Remove, hide (progressive disclosure), or merge.
3. Cut copy in half. Then again.
4. Stop when removing one more thing breaks comprehension or a decision. Put that one back.

| Cut | Keep |
|---|---|
| Intro paragraph restating the heading | A heading that says the answer |
| Helper text that restates the label | Helper text that answers a question the label raises |
| Competing buttons (`Save`, `Save & close`, `Save draft`, `Done`) | One primary, one escape |
| Eyebrows, section numbers, decorative meta strips, filler chips | Labels the user acts on |
| Marketing fluff, legalese, hedging in UI | The fact, the consequence, the action |
| Passive voice (`Changes will be saved`) | Active (`Save changes`) |
| Options nobody changes | A smart default |

**Never cut:** information a decision needs · accessible names · hierarchy (something must stand out) · real domain complexity (match the task, don't dumb it down). **Mystery ≠ minimalism.**

## 🌍 i18n-safe copy
Write once, translate everywhere. Mechanics in [i18n.md](i18n.md).

| ❌ Breaks in translation | ✅ Safe |
|---|---|
| `"You have " + n + " item" + (n>1?"s":"")` | Full message with plural key: `{count} items in cart` / `_plural` variants |
| `t("delete") + " " + name + "?"` (concatenated) | `t("confirm.delete", { name })` → `Delete "{name}"?` |
| Text baked into images/SVG | Live text layered on the image |
| Button sized to English (`Save`) | Allow +30–40% expansion (German, Finnish); no fixed-width buttons |
| Idioms, puns, sports metaphors | Literal phrasing (`Try again`, not `Take another swing`) |
| `MM/DD/YYYY`, `$`, hardcoded `,` decimals | Locale formatting ([dates-money-timezones.md](dates-money-timezones.md)) |
| Gendered pronoun for a variable user | Name or neutral (`Maya joined`, `they`) |
| Truncating with `…` in the string | CSS truncation + full value in `title`/tooltip |

- Variables as named placeholders so translators can reorder.
- Keep sentence case in source; locales handle their own casing.
- Test at 200% zoom and with the longest locale before ship.

## 🔎 Copy self-audit (before ship)
Re-read **every visible string**: headings, buttons, labels, captions, alt text, errors, toasts, footer, `aria-label`s.

- [ ] Voice matches the 3 voice words in `DESIGN.md`. No anti-words.
- [ ] No slop pattern from the table above. No lorem / Acme / John Doe / round fake stats.
- [ ] Each string grammatical, self-contained, no unclear referent.
- [ ] Every button = verb + object. One label per intent. Confirm buttons repeat the verb.
- [ ] Every error = what + recovery. No blame, no `Oops`, no raw code headline.
- [ ] Every empty state names its case + next action.
- [ ] Same term for the same concept everywhere (glossary).
- [ ] Nothing said twice on one screen.
- [ ] Works at 200% zoom, longest locale, longest real value.
- [ ] Unsure a line makes sense? Replace with a plain functional sentence. Boring beats cute-and-wrong.

## 📋 Paste-ready block (CLAUDE.md / DESIGN.md)
```markdown
## UX copy
- Voice: <3 words>. Never: <3 anti-words>. Derived from PRODUCT.md; differs from sibling apps.
- Tone shifts by moment: routine success quiet ("Saved."), errors calm, money/privacy/deletion zero humor.
- Glossary: one word per concept (list terms here). Sentence case everywhere.
- Real content only: locale-appropriate names, believable brands, organic numbers. No lorem/Acme/John Doe. Never invent customers, stats, prices, testimonials; label sample data.
- No slop: transform/reimagine/unlock/seamless/effortless/powerful, aphorism triplets, mock-poetic micro-meta, chat-assistant voice, em-dash separators, "!" on success, "Oops".
- Headline <=8 words, outcome not category. Subhead <=25 words, never restates. CTA verb+object <=3 words, one label per intent.
- Buttons name the outcome, not the gesture. Never Submit/OK/Yes/No alone.
- Errors: what failed + how to fix. No blame, no raw codes, keep user input, announce via aria-live.
- Destructive: prefer undo. Else name object + consequence + count; button repeats verb; safe option focused.
- Forms: label above, placeholder = example, requirements before submit, validate on blur, error under field.
- Empty states: name the case (first use/no results/filtered/no permission/failed/not yet) + one next action.
- Onboarding: name the activation moment; shortest path to it; skippable; sample data > tours; checklist only for 3-5 independent steps; never show twice.
- Distill: one job per surface; cut copy in half twice; stop when the next cut breaks a decision.
- i18n: no concatenation, named placeholders, plural keys, +40% expansion room, no idioms, no text in images.
- Before ship: re-read every visible string against this block.
```

---
Sources / further reading (as of 2026-09): [pbakaus/impeccable](https://github.com/pbakaus/impeccable) (`clarify`, `onboard`, `distill`, `shape`, `delight`, `PRODUCT.md`) · [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) (copy self-audit, content slop, CTA rules).
