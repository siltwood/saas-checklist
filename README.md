# Starter — Launch Checklists

Reusable checklists for taking a SaaS to production. Distilled from the lenamaps-product
launch (live since 2026-05) — its post-launch audit, incidents, and runbooks. Copy the
relevant files into a new project and work them top to bottom; items marked **[lesson]**
trace back to a real incident or audit finding.

## The checklists

| File | What it covers | When |
|---|---|---|
| [SAAS-PROD-CHECKLIST.md](SAAS-PROD-CHECKLIST.md) | The master list: secrets, auth, DB, billing, API hardening, frontend quality, testing/CI, environments, observability, deploy/rollback/DR, legal, launch day | Start here; work throughout the build |
| [GDPR-CHECKLIST.md](GDPR-CHECKLIST.md) | Data inventory, consent, privacy/cookie policy, delete/export rights, breach plan | Before public launch |
| [GOOGLE-CHECKLIST.md](GOOGLE-CHECKLIST.md) | GCP project, service-account keys, OAuth consent screen, GA4, Search Console | When wiring Google anything |
| [CLOUDFLARE-CHECKLIST.md](CLOUDFLARE-CHECKLIST.md) | DNS, TLS, WAF/rate limits, Workers, R2, Access/Tunnels, account security | When setting up the domain + edge |
| [EMAIL-LOCKDOWN-CHECKLIST.md](EMAIL-LOCKDOWN-CHECKLIST.md) | Gmail/Google account hardening + SPF/DKIM/DMARC + sender architecture | Day you buy the domain |
| [SOCIALS-CHECKLIST.md](SOCIALS-CHECKLIST.md) | Handle claiming, account security, profile consistency, launch wiring | Day you buy the domain |
| [SHARE-CARDS-CHECKLIST.md](SHARE-CARDS-CHECKLIST.md) | OG/Twitter meta, the 1200×630 image, per-route + dynamic cards, validators | Before anyone shares a link |

## Suggested order for a new project

1. **Day 0** (domain purchased): EMAIL-LOCKDOWN → CLOUDFLARE §1–2 → SOCIALS §1–2
2. **While building**: SAAS-PROD §1–9 as you go; GOOGLE when integrating; GDPR §1 (data inventory) early — it shapes the schema
3. **Pre-launch week**: GDPR, SHARE-CARDS, CLOUDFLARE §3 (WAF), SAAS-PROD §12 (launch day, in order)
4. **Post-launch**: SAAS-PROD §13, first-week watch items

## Ideas for future checklists

- Accessibility (a11y) pass
- Status page + public incident comms
- App Store / Play Store presence
- Pricing page copy + conversion basics
- SEO content/blog setup
