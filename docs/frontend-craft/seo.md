# 🔍 SEO

Make every public page discoverable, shareable, and fast. SEO is the sum of many small correct things — content, meta, structured data, sitemaps, media, and performance. Bake it into the template so it's automatic, not a launch-day scramble.

> ⚠️ **SEO (and SSR) is for public pages only.** Closed platforms behind a login get no SEO benefit — crawlers never authenticate — so those are plain **SPAs, no SSR** ([frontend-solidjs](../stack/frontend-solidjs.md#the-rule-ssr-is-only-for-seo)). Everything below applies to the public marketing/content site, not the app behind auth.

## Render content so crawlers see it
- **SSR/SSG for public content** ([SolidStart](../stack/frontend-solidjs.md)) — don't hide text behind client-only JS. The HTML response must contain the words you want ranked.
- **Semantic HTML** — one `<h1>` per page, ordered `<h2>/<h3>`, `<nav>/<main>/<article>/<footer>`, real `<a href>` links. Crawlers and screen readers both win.
- **Unique, descriptive content** per route — no thin/duplicate pages.

## Per-page meta (every route sets these)
```html
<title>Page Title — Brand (≤60 chars)</title>
<meta name="description" content="Compelling 150–160 char summary.">
<link rel="canonical" href="https://example.com/path">
<meta name="robots" content="index,follow">
<html lang="en">
```
Drive these from route data so each page is unique — never a global default title.

## Open Graph + Twitter (social cards)
```html
<meta property="og:type" content="website">
<meta property="og:title" content="…">
<meta property="og:description" content="…">
<meta property="og:image" content="https://example.com/og/path.png"> <!-- 1200×630 -->
<meta property="og:url" content="https://example.com/path">
<meta name="twitter:card" content="summary_large_image">
```
- OG image **1200×630**, optimized ([assets](assets-optimization.md)); can be generated per-page.
- Validate with each platform's debugger before launch.

## Structured data (JSON-LD)
Give search engines machine-readable meaning → rich results.
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "…",
  "datePublished": "2026-06-24",
  "author": { "@type": "Organization", "name": "Brand" },
  "image": "https://example.com/og/path.png"
}
</script>
```
Pick the type that fits: `Organization`/`WebSite` (home, + `SearchAction` sitelinks box), `Article`/`BlogPosting`, `Product` + `Offer`, `BreadcrumbList`, `FAQPage`, `Event`. Keep JSON-LD in sync with the visible content (mismatches get penalized).

## Sitemap + robots
```
# robots.txt
User-agent: *
Allow: /
Sitemap: https://example.com/sitemap.xml
```
- **Generate `sitemap.xml` at build** from the route table — every indexable URL with `lastmod`. Split into a sitemap index if >50k URLs.
- Submit it in Search Console / Bing Webmaster.
- `robots.txt` to disallow private/admin/staging; don't accidentally `Disallow: /` in prod.

## Media SEO
- **Descriptive `alt`** on every meaningful image; filenames that describe content.
- Include images/video in (or as) a sitemap; add `VideoObject` JSON-LD for video pages.
- Serve modern formats with correct dimensions ([assets-optimization](assets-optimization.md)) — image weight hurts ranking via slow LCP.

## Performance & Core Web Vitals
Speed is a ranking factor. Targets:
| Metric | Good |
|---|---|
| LCP (largest contentful paint) | < 2.5s |
| INP (interaction to next paint) | < 200ms |
| CLS (layout shift) | < 0.1 |

Levers: SSR + cached HTML, [optimized assets](assets-optimization.md), preconnect/preload critical resources, `font-display: swap`, set image dimensions, defer non-critical JS, [offload heavy work to workers](pwa-offline.md).

## Internationalized SEO
For multi-locale apps, emit `hreflang` alternates and locale-specific canonicals so the right language ranks per region → [i18n.md](i18n.md).
```html
<link rel="alternate" hreflang="es" href="https://example.com/es/path">
<link rel="alternate" hreflang="x-default" href="https://example.com/path">
```

## Make it automatic
- A shared `<Seo>` component / head helper takes `title`, `description`, `canonical`, `image`, `jsonLd` per route — no page ships without them.
- **Sitemap + robots generated in CI** from the route table.
- Lint/check in CI: every public route has a unique title + description; OG image resolves; JSON-LD validates.

## Checklist
- [ ] SSR/SSG for public pages, semantic HTML, one `<h1>`
- [ ] Unique `<title>` + meta description per route
- [ ] `canonical` + correct `robots`
- [ ] Open Graph + Twitter card + 1200×630 image
- [ ] JSON-LD for the page type, matching visible content
- [ ] `sitemap.xml` (auto) + `robots.txt` + submitted
- [ ] `alt` on all images; media in sitemap
- [ ] LCP/INP/CLS in budget
- [ ] `hreflang` if multi-locale
