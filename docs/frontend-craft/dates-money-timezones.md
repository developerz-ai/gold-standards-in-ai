# 🕒 Dates, Money & Timezones

Show dates in **each user's timezone**, money in **their locale & currency**. Never hardcode a format or assume a zone. Use `Intl` at the view layer (frontend) and `date-fns-tz` for scheduling logic (backend).

## Golden rule: store raw, format at the edge
| Concern | Store | Format |
|---|---|---|
| Instants | UTC (`Date` / ISO 8601) | `Intl.DateTimeFormat` with explicit `timeZone` |
| Money | integer **minor units** (cents) + currency code | `Intl.NumberFormat` with `style: "currency"` |
| Locale / tz | on the user record | passed **explicitly** into every formatter |

Formatting is a presentation detail. The same instant renders differently for two users — so only convert when you display.

## Dates in the user's timezone
Pass the member's `tz` (an IANA name like `Europe/Berlin`) to `Intl.DateTimeFormat`. Use `timeZoneName: "shortOffset"` + `formatToParts` to surface the offset:
```ts
type Member = { tz: string; locale: string };

export function renderForMember(at: Date, member: Member): string {
  const fmt = new Intl.DateTimeFormat(member.locale, {
    timeZone: member.tz,
    dateStyle: "medium",
    timeStyle: "short",
    timeZoneName: "shortOffset",
  });

  const parts = fmt.formatToParts(at);
  const offset =
    parts.find((p) => p.type === "timeZoneName")?.value ?? "";
  const text = parts
    .filter((p) => p.type !== "timeZoneName")
    .map((p) => p.value)
    .join("")
    .trim();

  return `${text} (${offset})`;
  // → "14 Mar 2026, 09:00 (GMT+1)" for Europe/Berlin
  // → "14 Mar 2026, 04:00 (GMT-4)" for the same instant in America/New_York
}
```

### Locale-aware long dates
```ts
export function longDate(at: Date, locale: string): string {
  return new Intl.DateTimeFormat(locale, {
    day: "numeric",
    month: "long",
    year: "numeric",
    timeZone: "UTC",
  }).format(at);
  // en-US: "November 5, 2011"  ·  de-DE: "5. November 2011"  ·  pt-BR: "5 de novembro de 2011"
}
```
**English ordinal special case:** `Intl` gives `"November 5, 2011"`, not `"5th of November 2011"`. If a design calls for the ordinal form, build it explicitly with `Intl.PluralRules`:
```ts
const ordinals: Record<Intl.LDMLPluralRule, string> = {
  one: "st", two: "nd", few: "rd", other: "th",
  zero: "th", many: "th",
};
function ordinal(n: number, locale = "en") {
  const suffix = ordinals[new Intl.PluralRules(locale, { type: "ordinal" }).select(n)];
  return `${n}${suffix}`; // 1 → "1st", 5 → "5th", 22 → "22nd"
}
// `${ordinal(5)} of November 2011` → "5th of November 2011"
```

## Backend scheduling across DST
Computing "send at 09:00 **local** tomorrow" is where naive math breaks: DST gaps (a wall-clock time that doesn't exist), overlaps (a time that happens twice), and half-hour/45-min offsets. `date-fns-tz` resolves these correctly — never add `offset * 3600` by hand.
```ts
import { formatInTimeZone, fromZonedTime } from "date-fns-tz";

type Schedule = { tz: string; hour: number; minute: number };

/** Next instant (UTC) matching the user's local HH:mm, strictly after `now`. */
export function nextLocalSlot(schedule: Schedule, now: Date): Date {
  const { tz, hour, minute } = schedule;
  const pad = (n: number) => String(n).padStart(2, "0");

  for (let dayOffset = 0; dayOffset < 3; dayOffset++) {
    // wall-clock date in the user's tz, dayOffset days ahead
    const ymd = formatInTimeZone(
      new Date(now.getTime() + dayOffset * 86_400_000),
      tz,
      "yyyy-MM-dd",
    );
    // interpret that local wall-clock time -> a UTC instant (DST-aware)
    const candidate = fromZonedTime(`${ymd}T${pad(hour)}:${pad(minute)}:00`, tz);
    if (candidate.getTime() > now.getTime()) return candidate;
  }
  throw new Error("no slot found");
}
```
`fromZonedTime` maps a local wall-clock string in a zone to the correct UTC instant; `formatInTimeZone` renders an instant in a zone. Both account for the active DST rules on that date, so 09:00 stays 09:00 to the user across the spring-forward / fall-back boundary.

## Money
Store amounts as **integer minor units** (cents, pence, yen has none) to avoid binary-float drift (`0.1 + 0.2 !== 0.3`). Keep the **currency code with the amount** — never assume one currency.
```ts
type Money = { minor: number; currency: string }; // { minor: 129900, currency: "EUR" }

export function formatMoney(m: Money, locale: string): string {
  return new Intl.NumberFormat(locale, {
    style: "currency",
    currency: m.currency,
  }).format(m.minor / 100);
  // de-DE / EUR → "1.299,00 €"   ·   en-US / USD → "$1,299.00"
}
```
- **Math on `minor` only** (integers); divide by the minor-unit scale solely at format time.
- `Intl` handles grouping, decimal separators, symbol placement, and currency-specific decimals (JPY shows no cents) — don't reimplement.
- Multi-currency: store `{ minor, currency }` as a unit; never sum across currencies without an explicit conversion step.

## Agent datetime tool
LLM agents have no clock and no timezone. Expose "current datetime for a zone" as a tool so they can reason about time:
```ts
export const nowTool = {
  name: "current_datetime",
  description: "Current date and time in a given IANA timezone.",
  input: { timeZone: "string" }, // e.g. "Asia/Kolkata"
  run: ({ timeZone }: { timeZone: string }) => ({
    iso: new Date().toISOString(),
    local: new Date().toLocaleString("en-US", { timeZone }),
    timeZone,
  }),
};
```
See [../ai-agents/agent-sdk.md](../ai-agents/agent-sdk.md) for wiring tools into an agent.

## Rules
- **Store UTC / minor units.** Persist instants in UTC and money as integers.
- **Format at the view layer** with `Intl` — never store formatted strings.
- **Pass user tz + locale explicitly** into every formatter; no ambient defaults.
- **Test DST transitions** — spring-forward gap, fall-back overlap, and a non-hour offset zone.
- Locale resolution (normalize tags, fall back to default) lives in [i18n.md](i18n.md).
