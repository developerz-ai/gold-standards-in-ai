# 🛡️ CAPTCHA (hCaptcha) + AI-Testable Forms

Stop bots without breaking automation. hCaptcha needs no wrapper library on the frontend, must always be verified server-side, and must leave an env-gated escape hatch so AI agents can still drive your forms in test.

## Frontend: load once, render two lines
The widget is raw DOM. Load the script idempotently, then render a `<div class="h-captcha">`. No framework binding required.

```ts
// load-hcaptcha.ts — resolves once the global is ready, safe to call N times
let scriptPromise: Promise<void> | null = null;

export function loadHcaptchaScript(): Promise<void> {
  if (typeof window === "undefined") return Promise.resolve();
  if ((window as any).hcaptcha) return Promise.resolve();
  if (scriptPromise) return scriptPromise;

  scriptPromise = new Promise((resolve, reject) => {
    const s = document.createElement("script");
    s.src = "https://js.hcaptcha.com/1/api.js?render=explicit";
    s.async = true;
    s.defer = true;
    s.onload = () => resolve();
    s.onerror = () => {
      scriptPromise = null; // allow retry on next call
      reject(new Error("Failed to load hCaptcha"));
    };
    document.head.appendChild(s);
  });
  return scriptPromise;
}
```

The widget itself is declarative. hCaptcha auto-renders any `.h-captcha` element after the script loads:

```html
<div
  class="h-captcha"
  data-sitekey="YOUR_SITE_KEY"
  data-theme="dark"
></div>
```

### A Solid component (framework-agnostic pattern)
`onMount` loads the script; the markup is the same two lines. Works the same in any framework — swap `onMount` for `useEffect` / `mounted`.

```tsx
import { onMount, createSignal } from "solid-js";
import { loadHcaptchaScript } from "./load-hcaptcha";

export function Captcha(props: { sitekey: string; theme?: "light" | "dark" }) {
  const [failed, setFailed] = createSignal(false);

  onMount(() => {
    loadHcaptchaScript().catch(() => setFailed(true));
  });

  return (
    <>
      <div class="h-captcha" data-sitekey={props.sitekey} data-theme={props.theme ?? "dark"} />
      {failed() && <p class="captcha-error">Could not load verification. Refresh to retry.</p>}
    </>
  );
}
```

No state binding, no token plumbing — hCaptcha writes the token into a hidden input for you (next section).

## Form integration: the hidden input
On solve, hCaptcha injects a hidden `<input name="h-captcha-response">` into the surrounding form. Read it with `FormData` — no JS callback needed.

```ts
async function onSubmit(form: HTMLFormElement) {
  const data = new FormData(form);
  const token = data.get("h-captcha-response") as string | null;
  if (!token) return showError("Please complete the verification.");

  await fetch("/api/login", {
    method: "POST",
    body: JSON.stringify({
      email: data.get("email"),
      password: data.get("password"),
      captchaToken: token,
    }),
    headers: { "content-type": "application/json" },
  });
}
```

## Backend verification (Bun / Node)
**Always verify server-side.** POST the secret + token to the siteverify endpoint. Use `AbortSignal.timeout` so a slow/hung hCaptcha never wedges your request. **Fail closed** — any error means "not verified".

```ts
// verify-hcaptcha.ts
const HCAPTCHA_SECRET = process.env.HCAPTCHA_SECRET!;
const VERIFY_URL = "https://api.hcaptcha.com/siteverify";

export async function verifyHcaptcha(token: string, remoteIp?: string): Promise<boolean> {
  if (!token) return false;

  const body = new URLSearchParams({ secret: HCAPTCHA_SECRET, response: token });
  if (remoteIp) body.set("remoteip", remoteIp);

  try {
    const res = await fetch(VERIFY_URL, {
      method: "POST",
      headers: { "content-type": "application/x-www-form-urlencoded" },
      body,
      signal: AbortSignal.timeout(5000), // never hang on a flaky upstream
    });
    if (!res.ok) return false;
    const data = (await res.json()) as { success?: boolean };
    return data.success === true;
  } catch {
    return false; // fail closed: network error, timeout, bad JSON → not verified
  }
}
```

| Field | Sent | Notes |
|---|---|---|
| `secret` | required | server-only key, never shipped to client |
| `response` | required | the `h-captcha-response` token |
| `remoteip` | optional | caller IP for extra signal |

## 🤖 AI-testability escape hatch
CAPTCHA is **anti-bot, not auth.** It exists to slow down automated abuse — it does *not* establish who you are. That makes it safe to bypass *verification only* for trusted test automation, while every real security control still runs.

The problem: Playwright / [MCP](../ai-agents/tools-and-mcp.md) agents can't solve hCaptcha, so they can't test login. The fix: a `?debug-ai=true` query param that skips **captcha verification only**, gated behind an env flag so it is **OFF in production**.

What the bypass does *not* touch:
- The widget **still renders** → no DOM drift, tests see the real page.
- Credentials are **still checked** → wrong password still fails.
- Rate limits and account lockouts **still apply**.
- Auth, sessions, CSRF — all unchanged.

```ts
// inside the login handler
const ALLOW_AI_DEBUG_LOGIN = process.env.ALLOW_AI_DEBUG_LOGIN === "true";

async function checkCaptcha(req: Request, token: string): Promise<boolean> {
  const url = new URL(req.url);
  const debugBypass = ALLOW_AI_DEBUG_LOGIN && url.searchParams.get("debug-ai") === "true";

  // escape hatch: skip CAPTCHA verification only — auth still runs below
  if (debugBypass) return true;

  return verifyHcaptcha(token, req.headers.get("x-forwarded-for") ?? undefined);
}

// usage
async function login(req: Request, creds: Credentials, token: string) {
  if (!(await checkCaptcha(req, token))) {
    return json({ error: "captcha_failed" }, 400);
  }
  // credentials, rate limit, lockout, session — all still enforced
  return authenticate(creds);
}
```

Why this is the right shape: the widget keeps rendering so your test exercises the exact production UI; only the unsolvable-by-machine step is short-circuited. See [make your app AI-debuggable](../writing-for-agents/memory-and-mcp.md) for the broader pattern, and [Playwright / tool access](../ai-agents/tools-and-mcp.md) for how agents drive the browser.

## Rules
- **Verify server-side, always.** The client token is a claim, not proof.
- **Fail closed.** Timeout, network error, malformed response → treat as not verified.
- **Never trust the client.** A present `h-captcha-response` means nothing until siteverify confirms it.
- **Keep the secret server-only.** Site key is public; secret is not.
- **Env-gate the bypass and keep it OFF in prod** (`ALLOW_AI_DEBUG_LOGIN` unset/false). Bypass skips *verification only*, never auth.
- **Always `AbortSignal.timeout`** the verify call so a flaky upstream can't hang requests.
