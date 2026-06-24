# 🗜️ Asset Optimization

Ship the smallest bytes that still look great. Optimize images, video, and fonts **before they enter version control** — never commit a 4 MB PNG and "fix it later." Fast assets = fast pages = good [Core Web Vitals](seo.md) = better DX for everyone (and smaller repos).

## Images — modern formats first
| Format | Use |
|---|---|
| **AVIF** | best compression — first choice for photos |
| **WebP** | excellent + universal — the safe default |
| JPEG/PNG | fallback only, or PNG for crisp UI/transparency |
| **SVG** | icons, logos, line art (then minify with SVGO) |

Serve modern formats with a fallback and let the browser pick:
```html
<picture>
  <source srcset="/img/hero.avif" type="image/avif">
  <source srcset="/img/hero.webp" type="image/webp">
  <img src="/img/hero.jpg" alt="…" width="1200" height="630" loading="lazy" decoding="async">
</picture>
```
- **Always set `width`/`height`** (or `aspect-ratio`) → no layout shift (CLS).
- **`loading="lazy"`** below the fold; eager + `fetchpriority="high"` for the LCP image.
- **Responsive `srcset`** with real widths so phones don't download desktop pixels.
- Strip EXIF/metadata on export.

## Video
- Encode **H.264/AAC MP4** (universal) + **WebM/VP9 or AV1** (smaller) with `<source>` fallback.
- `preload="metadata"`, a `poster` image, and never autoplay with sound.
- Long/often-watched media → store in [R2](../stack/databases.md) and stream (HLS), don't commit it.
- For generated media, see [../ai-agents/media-generation.md](../ai-agents/media-generation.md).

## Fonts
- **Self-host `woff2`** (smallest, universal) — don't hotlink third-party font CDNs (privacy + a render-blocking round trip).
- **Subset** to the glyphs/languages you actually use (`fonttools`/`subfont`) — often 80%+ smaller.
- `font-display: swap` so text paints immediately; `<link rel="preload" as="font" crossorigin>` the critical face.
- Prefer 1–2 families, a couple of weights. A variable font can replace many static weights.
```css
@font-face {
  font-family: "Inter";
  src: url("/fonts/inter-var.woff2") format("woff2");
  font-weight: 100 900;
  font-display: swap;
}
```

## Optimize BEFORE version control
The rule: **assets are committed already optimized.** Enforce it with a script, not discipline.

```bash
# scripts/assets/optimize.ts (Bun)  — run on staged assets, fail if not optimized
# images → AVIF + WebP (sharp), strip metadata, generate srcset widths
# svg    → SVGO
# fonts  → subset + woff2
# video  → ffmpeg to mp4 + webm (or reject big raw files)
bun scripts/assets/optimize.ts assets/
```
- Wire it as a **pre-commit hook** so unoptimized assets physically can't land ([hooks](../writing-for-agents/hooks-and-permissions.md)).
- Or generate derivatives **at build time** (Vite image plugins) and commit only the source — pick one model and document it in `CLAUDE.md`.
- **Block giant binaries in CI** (warn/fail over a size budget); large originals belong in R2, not git.
- Keep one **lossless source** per asset; commit the optimized deliverables (or build them).

## Budgets
| Asset | Soft budget |
|---|---|
| Hero/LCP image | ≤ 150 KB (AVIF/WebP) |
| Inline icon (SVG) | ≤ 5 KB |
| Web font (per face, subset) | ≤ 30 KB |
| Any single committed binary | flag > 1 MB |

Tie it to the [perf targets](seo.md#performance--core-web-vitals): if the page can't hit a good LCP, the assets are too heavy.
