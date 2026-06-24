# 🎨 Frontend Craft

Cross-cutting concerns that make a UI feel professional and work for real users everywhere. Stack-agnostic patterns (examples in SolidJS) you'll want in nearly every app.

| File | Topic |
|---|---|
| [i18n.md](i18n.md) | 🌍 Internationalization |
| [theming-dark-mode.md](theming-dark-mode.md) | 🌗 Auto theme & dark mode with CSS variables |
| [dates-money-timezones.md](dates-money-timezones.md) | 🕰️ Per-user timezones · 💰 currency formatting |
| [captcha.md](captcha.md) | 🛡️ hCaptcha + keeping the app AI-testable |
| [css-scss-craft.md](css-scss-craft.md) | 💅 CSS3 + SCSS showcase: animations, gradients, tokens, modern CSS |
| [pwa-offline.md](pwa-offline.md) | 📲 PWA, responsive, web workers & offline-first |
| [assets-optimization.md](assets-optimization.md) | 🗜️ Optimized images/video/WebP & fonts, optimize-before-commit |
| [seo.md](seo.md) | 🔍 SEO: meta, Open Graph, JSON-LD, sitemap, media, Core Web Vitals |

## 👀 Let the agent SEE the UI — Playwright MCP (headless)
A coding agent can't make a web app *look nice* if it's working blind. Give it eyes: wire up [microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) (headless) so the agent can open the running app, navigate, screenshot, read the accessibility tree, and iterate on the visual result.
```bash
claude mcp add playwright -- npx @playwright/mcp@latest --headless
```
- The loop becomes **change → screenshot → compare → refine**, not change → hope.
- Headless runs fine on a [Linux dev VPS](../developer-experience/dev-vps.md) with no display.
- Pair with an [AI-debug captcha bypass](captcha.md) so the agent can reach authed pages.
- More on driving browsers as a tool: [../ai-agents/tools-and-mcp.md](../ai-agents/tools-and-mcp.md).

## The baseline every app ships with
- **i18n from day one** — even if it's one locale, structure for many.
- **Auto dark mode** — respect `prefers-color-scheme`, allow an override, persist it.
- **Locale- and timezone-correct dates & money** — never hardcode a format or assume a timezone.
- **CAPTCHA where bots hit you** — with an AI-debug escape hatch so [agents can still test the app](../writing-for-agents/memory-and-mcp.md#make-your-app-ai-debuggable).
- **Design tokens** — semantic CSS variables, not hardcoded colors.
- **Responsive + installable** — mobile-first fluid layouts; PWA, web workers, and offline-first where the use case warrants → [pwa-offline.md](pwa-offline.md).
- **Optimized assets** — WebP/AVIF images, compressed video, subset fonts, optimized *before* commit → [assets-optimization.md](assets-optimization.md).
- **SEO baked in** — unique meta, Open Graph, JSON-LD, auto sitemap, fast Core Web Vitals → [seo.md](seo.md).
