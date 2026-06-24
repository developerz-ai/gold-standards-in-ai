# 🗜️ Compressed Config

Files the agent reads every session cost tokens **forever**. Write them compressed — same signal, fewer words.

| File | Read when | Cost |
|---|---|---|
| `CLAUDE.md` | every session start | every turn |
| `SKILL.md` | skill activates | per activation |
| `agents/*.md` | subagent spawns | per spawn |
| `commands/*.md` | command runs | per invocation |

## The 10 rules
1. **Lead with the rule, not the reason.** `Business logic in src/services/.` not `It's important that…`
2. **Fragments over sentences.** `Tests: bun test. Lint: bin/lint.`
3. **Tables > paragraphs** for any structured data.
4. **File paths > descriptions.** `see src/services/auth/`.
5. **Drop filler:** articles, `just`, `really`, `basically`, `in order to`, `it's important to note`.
6. **No meta-framing.** Skip "This section covers…".
7. **No rhetoric.** Drop "this is critical" — show it in the rule.
8. **Code/commands/paths unchanged.** Compress prose only.
9. **One rule per line** in rule lists.
10. **Why only when non-obvious.**

## Value-words — the signal you want
| Function | Words |
|---|---|
| Hard rule | `MUST`, `NEVER`, `ALWAYS`, `required`, `forbidden` |
| Cause/cost | `breaks when`, `fails if`, `causes`, `without this` |
| Boundary | `before X`, `after Y`, `only in`, `not in` |
| Trigger | `when Z`, `if user asks`, `on commit` |

A section with zero value-words is probably narration — rewrite as instruction. Kill flow-words: `however`, `moreover`, `furthermore`, `thus`, `therefore`, `as such`.

## Open with a constraint, not a tour
| ❌ Tour | ✅ Constraint |
|---|---|
| "This is a Rails app using PostgreSQL. We follow service-oriented patterns…" | "Business logic → `app/services/<domain>/`. Controllers thin. Custom errors only." |

## Gap ≠ error
| Weak (gap) | Strong (error) |
|---|---|
| "We haven't documented webhooks." | "Webhooks: always call `Webhook.verify` first — signature check was missed twice." |
| "Auth is complex." | "Auth: never read `current_user` in models. Pass it explicitly." |

## Before / after
**Before** (~90 words of prose):
```markdown
## Architecture
We follow a service-oriented architecture pattern. All business logic
should be placed in service objects under app/services/, organized by
domain. Controllers should be kept thin...
```
**After** (~30 words, same info):
```markdown
## Architecture
- Business logic → `app/services/<domain>/`
- Controllers: thin. Parse → service → render.
- Models: persistence only.
```

## What NOT to compress
Leave verbatim — compression loses information: code blocks, shell commands, file paths, URLs/versions/env-var names, regex/queries/config values, error strings you match against.

## Size targets
`CLAUDE.md` < 600 · `SKILL.md` < 80 · agent < 50 · command < 30 lines.

## Self-check before committing a config file
- [ ] Opens with a constraint/rule, not history
- [ ] Every list item ≤15 words
- [ ] Tables used for any ≥3-row data
- [ ] Commands quoted exactly, not described
- [ ] Every section has ≥1 value-word
- [ ] Flow-words removed
- [ ] Under the size target

## Compression is not cryptic
Terse ≠ unreadable. Target **high info density**, not low word count. The breath test: can a new engineer read a section in one breath and know what to do? If not, add structure back.
