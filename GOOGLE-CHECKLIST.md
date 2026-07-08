# Google Setup Checklist

Everything Google a SaaS launch touches: Cloud project, OAuth, Analytics, Search Console,
service accounts. Items marked **[lesson]** come from a real production launch.

## 1. Google Cloud project hygiene

- [ ] One GCP project per product (not "My First Project" holding everything)
- [ ] Billing alerts set (budget + email alert at 50/90/100%)
- [ ] Only the APIs you use are enabled; check quota limits for anything user-facing
- [ ] 2FA/passkey on the owning Google account (see EMAIL-LOCKDOWN checklist)

## 2. Service accounts & keys

- [ ] One service account per purpose, least-privilege role (e.g. GA4 read-only for reporting)
- [ ] **Key JSON files never live in the repo working tree** — gitignore pattern (`*-credentials.json`) is not enough; the file leaks when the folder is zipped/shared. Store outside the project or in a secret manager **[lesson: `ga-credentials.json` sat in repo root]**
- [ ] Prefer short-lived/workload identity over downloaded keys where the platform supports it
- [ ] Rotation reference written: where each key is used, how to regenerate, what breaks if revoked
- [ ] Audit periodically: delete unused service accounts and stale keys

## 3. OAuth ("Sign in with Google")

- [ ] OAuth consent screen configured: app name, logo, support email, privacy policy + terms URLs (required for verification)
- [ ] Publishing status moved from "Testing" to "In production" (Testing caps you at 100 users and expires refresh tokens after 7 days)
- [ ] Scopes minimal (email + profile only unless you truly need more — sensitive scopes trigger a slow verification process)
- [ ] Authorized redirect URIs cover prod domain, localhost, and tunnel domains — and match your auth provider's callback exactly
- [ ] Test the full round-trip on production: sign up with Google → sign out → sign back in → incognito **[lesson: provider configured but redirect allowlist initially missing]**
- [ ] Branding verification submitted if you want the app name (not the domain) on the consent screen

## 4. Google Analytics 4

- [ ] GA4 property created; Measurement ID in the frontend (public by design — fine to commit)
- [ ] **Consent Mode v2, default-deny** — GA only fires after cookie consent; IP anonymization on; Google Signals off unless needed **[lesson]**
- [ ] Internal traffic excluded: filter out your own region/city or use an internal-traffic rule, or your dashboards are just you **[lesson: daily report excludes operator's cities]**
- [ ] Key events (conversions) defined: signup, checkout started, subscription completed
- [ ] Data retention setting bumped from the 2-month default to 14 months
- [ ] Data API access for reporting: service account added as a property viewer (for automated daily reports)
- [ ] GA4 property access restricted — remove default broad access, add only real operators

## 5. Google Search Console

- [ ] Domain property verified (DNS TXT record — verifies all subdomains + protocols at once)
- [ ] Sitemap submitted (`sitemap.xml` — which your repo must actually have; see SAAS-PROD-CHECKLIST §6)
- [ ] Check indexing after launch: `site:yourdomain.com`, request indexing for key pages
- [ ] Core Web Vitals + mobile usability reports reviewed after first crawl
- [ ] Email alerts on: manual actions, security issues, coverage errors

## 6. Misc Google surfaces (as applicable)

- [ ] reCAPTCHA/Turnstile decision made for public forms (prefer Cloudflare Turnstile if already on CF — no Google dependency)
- [ ] Google Workspace vs personal Gmail decision for ops/support email (see EMAIL-LOCKDOWN checklist)
- [ ] If you send email via Gmail SMTP: App Password hygiene — treat it like any secret, rotate on any leak **[lesson: leaked App Password re-rotated]**
- [ ] Google Business Profile only if you have a physical/local presence — skip otherwise
