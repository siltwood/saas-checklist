# Meta / Share Card Checklist

Make every link to your product unfurl beautifully — in social feeds, Slack, Discord, iMessage,
WhatsApp. Small effort, outsized first-impression payoff.

## 1. The tags (in `<head>`)

- [ ] `<title>` — brand + value prop, under ~60 chars
- [ ] `<meta name="description">` — under ~160 chars, reads like a sentence not a keyword pile
- [ ] `<link rel="canonical" href="https://yourdomain.com/">`
- [ ] Open Graph complete: `og:type`, `og:url`, `og:site_name`, `og:title`, `og:description`, `og:image`, `og:image:width`, `og:image:height`, `og:image:alt`
- [ ] Twitter/X: `twitter:card` = `summary_large_image`, `twitter:title`, `twitter:description`, `twitter:image` (X also reads OG tags, but set the card type explicitly)
- [ ] All URLs **absolute** (`https://…`), not relative — scrapers don't resolve relative paths
- [ ] `theme-color` meta (tints mobile browser chrome + some embeds)

## 2. The image

- [ ] **1200×630** PNG/JPG (1.91:1) — the one size that works everywhere; keep it under ~1 MB (some scrapers time out on 5 MB+)
- [ ] Designed for the crop: key content in the center ~1000×524 safe zone (platforms crop edges and corners round)
- [ ] Legible as a thumbnail — big product name, one visual, high contrast; no paragraph text
- [ ] Not just the logo on white — show the product (screenshot, map, output)
- [ ] Static file in `public/` served from your own domain (no redirects, no auth, correct `Content-Type`)
- [ ] Square fallback (~1200×1200) if you care about WhatsApp/iMessage small-tile rendering

## 3. Per-route cards (SPA gotcha)

- [ ] Decide which routes need their own card: `/`, `/pricing`, blog posts, public share links
- [ ] SPAs serve the same static `index.html` meta for every route — scrapers don't run JS. If per-route cards matter, render meta at the edge (Worker HTML rewrite), prerender, or SSR
- [ ] **User-generated share links**: dynamic OG title/description per shared item is a major virality upgrade (e.g. "Trip: NYC → Sydney") — consider dynamic OG image generation (Workers/`@vercel/og`-style, or a templated static fallback)
- [ ] Share URLs are short and clean (KV-backed short codes beat 2000-char query strings in every chat app)

## 4. Validate (before launch — caches are sticky)

- [ ] Facebook Sharing Debugger (developers.facebook.com/tools/debug) — also the only way to bust FB's scrape cache
- [ ] X/Twitter card validator (or just post the link in a DM to yourself)
- [ ] LinkedIn Post Inspector (linkedin.com/post-inspector)
- [ ] Paste the URL in real clients: Slack, Discord, iMessage, WhatsApp, Telegram — each has quirks the validators miss
- [ ] Test og-image URL directly in an incognito browser (no auth wall, no 302 chain)
- [ ] After changing the card: bump the image filename or add a query param — every platform caches unfurls aggressively

## 5. Bonus surfaces

- [ ] Favicon set: `.ico` + PNG sizes + `apple-touch-icon` (180×180) — shows in bookmark/unfurl contexts too
- [ ] `manifest.webmanifest` with icons if you want installability/Android polish
- [ ] oEmbed only if you're a content platform — otherwise skip
- [ ] Structured data (JSON-LD `Organization` + `WebSite`) for Google rich results — 10 minutes, do it
