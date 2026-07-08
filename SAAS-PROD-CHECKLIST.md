# Production SaaS Readiness Checklist

A reusable, stack-agnostic checklist for taking a SaaS from "works on my machine" to production.
Distilled from a real SaaS launch — both what it did right and the
gaps found in a post-launch audit. Copy this file into any new project and work it top to bottom.

Legend: items marked **[lesson]** come from a real production incident or audit finding.

---

## 1. Secrets & configuration

- [ ] All prod secrets live in the platform secret store (Wrangler secrets, Vercel env, Vault…) — never in committed config files
- [ ] `.env`, `.dev.vars`, `*credentials*.json` are gitignored AND verified never committed (`git log --all --diff-filter=A -- <file>`, `git ls-files`)
- [ ] `.env.example` documents every key name (no values) — this is your onboarding + rotation reference
- [ ] Keep real keys **out of the working tree**, not just out of git — a service-account JSON in the repo root leaks the moment the folder is zipped or shared **[lesson]**
- [ ] Written per-secret reference doc: what each secret does, where it's sourced, how to rotate it
- [ ] Count your secrets and verify after every push: `wrangler secret list` (or equivalent) shows the full expected set
- [ ] **Rotate any secret that has ever been pasted into a chat/LLM transcript** — transcripts are permanent records **[lesson]**
- [ ] When piping a secret into a CLI, verify the prompt text didn't leak into the stored value (the "paste " prefix bug) **[lesson]**
- [ ] Security-critical behavior must **fail closed by default**, not only when `ENVIRONMENT=production` is set — a deploy that forgets the env var must not silently downgrade to decode-only auth **[lesson]**
- [ ] Rotation drill: for each credential pair (e.g. Cloudflare Access id+secret), rotate both halves from the same rotation event and update every consumer in one pass

## 2. Auth

- [ ] Server verifies JWT **signature** (not decode-only), checks `exp`, rejects unknown algs (`alg: none`)
- [ ] Support both current and previous signing keys during a key rotation window; delete the legacy path after tokens expire
- [ ] Ghost-user defense: server re-checks the user still exists before acting on a valid token (handles deleted accounts)
- [ ] Auth provider Site URL + redirect-URL allowlist set for prod domain, localhost, and tunnel domains
- [ ] OAuth providers configured AND their redirect flow tested end-to-end (sign up, sign out, sign back in, incognito)
- [ ] Email templates customized (confirmation, reset, magic link) — sender name/address correct in every place it's configured
- [ ] Session validation on mount + tab-visibility change (catch revoked/expired sessions in long-lived tabs)
- [ ] Never trust `userId`/`email` from the request body on authenticated endpoints — derive identity from the verified token only

## 3. Database

- [ ] RLS (or equivalent) on **every** user table: users can only read/write their own rows — verify visually in the dashboard, not just in migration files
- [ ] Billing/subscription tables are user-**read-only**; only the service role writes them (never store plan in user-mutable metadata)
- [ ] Tier/size limits enforced at the DB layer (triggers/constraints), not only in the client — defense in depth
- [ ] Hard ceiling on user-writable row size (`octet_length` / array caps) on any table the client writes directly
- [ ] `SECURITY DEFINER` functions: revoke default `PUBLIC EXECUTE`, pin `search_path` **[lesson: real privilege-escalation exploit]**
- [ ] Migrations are numbered, committed, and applied through one tool — **no manual prod hotfixes that get back-committed later** (schema drift) **[lesson]**
- [ ] GDPR RPCs: `delete_own_account()` and `export_own_account()`, surfaced in account settings, deletions logged
- [ ] Know your delete semantics (hard-delete cascade vs soft-delete) and make reporting queries match

## 4. Billing (Stripe)

- [ ] Webhook signature verified (HMAC + timestamp tolerance for replay protection); use a **constant-time compare** **[lesson]**
- [ ] Webhook idempotency: dedupe by event id (KV/unique index) so retries can't double-apply
- [ ] Webhook tolerates deleted users (FK violation → 200 no-op, or Stripe retries forever) **[lesson]**
- [ ] `livemode` guard: prod webhook drops test-mode events
- [ ] All lifecycle events handled: `checkout.session.completed`, `invoice.paid`, `invoice.payment_failed`, `customer.subscription.updated/deleted`, `charge.refunded`
- [ ] **Refund → auto-downgrade path exists and is tested**
- [ ] Plan-change flows (upgrade proration, downgrade at period end) tested both directions; canceling releases any subscription schedule first **[lesson: "managed by schedule" rejection]**
- [ ] All price strings come from one constants file; price IDs are secrets/env, never hardcoded
- [ ] Test accounts seeded for every tier (script it: `seed --seed-users`) so manual QA is repeatable
- [ ] Launch sequence: deploy on **test keys** → full smoke (4242 card, webhook → DB row, cancel, plan switch) → flip to live keys → **subscribe with your own real card** → refund yourself and confirm the downgrade webhook lands
- [ ] Expired-subscription UX: over-limit banner + re-subscribe path (don't strand users' data, don't silently delete)

## 5. API hardening

- [ ] Strict input validation on every endpoint: regex/range checks on params, enum allowlists, length caps
- [ ] Rate limiting at the edge (WAF rule) AND/OR per-IP in the app — know that KV-based limiting is best-effort and fails open; treat it as cost protection, not a security control
- [ ] Sanitized error responses: map internal errors to client-safe messages; never echo upstream hostnames/stack traces **[lesson]**
- [ ] CSP headers on every response; `frame-ancestors 'none'` (clickjacking protection on auth/checkout)
- [ ] Open-redirect protection: validate `returnUrl` against a domain allowlist
- [ ] Timeouts + abort on every upstream fetch
- [ ] Internal services fronted by service tokens / private tunnels — nothing listens on a public port it doesn't need (bind 127.0.0.1, firewall the rest)
- [ ] CORS: if SPA and API are same-origin you may need none — but write that assumption down; it breaks silently if the architecture changes

## 6. Frontend production quality

- [ ] React ErrorBoundary wrapping the app with a recoverable fallback ("Try again" / "Reload")
- [ ] Global error capture (`window.onerror` + `unhandledrejection`) reporting to your own endpoint, with:
  - [ ] PII redaction (emails, UUIDs, constraint values) before sending
  - [ ] client-side rate limit (e.g. 5/min) and an ignore-list for benign browser noise
  - [ ] dev-origin suppression (localhost/tunnels don't pollute prod error data)
- [ ] Stale-chunk resilience: lazy imports auto-reload once on post-deploy chunk-load failure; deployed-version detector refreshes idle tabs **[lesson]**
- [ ] 404 catch-all route (`path="*"`)
- [ ] Bundle strategy: vendor `manualChunks`, route-level lazy loading for secondary pages, check the final bundle size (one audited app shipped a single 1.9 MB JS file — don't)
- [ ] Large binary assets on CDN/object storage, not in `public/` (they get copied into every build)
- [ ] SEO: title/description/canonical, full OG + Twitter card set with a real og-image, `robots.txt` **with** a `Sitemap:` directive, `sitemap.xml`, per-route meta for marketing pages
- [ ] Favicon set complete (ico + PNG sizes + apple-touch-icon + `theme-color`)
- [ ] Loading/empty states for every async surface; toasts with dedup + auto-dismiss
- [ ] First-run onboarding (dismissible, localStorage-flagged) and a welcome flow scoped per user id (burner-email re-signups get welcomed again) **[lesson]**
- [ ] Feature flags in one constants file; DEBUG tied to dev builds only
- [ ] Dark mode (if offered) via tokens, tested in exports/screenshots too

## 7. Testing & CI

- [ ] Unit/component tests colocated with features; services and utils covered
- [ ] E2E smoke suite (Playwright): app loads, core flow, auth pages, legal pages, share/deep links
- [ ] **CI pipeline exists** — tests + build gate every push; deploy is not "npm run deploy from whichever laptop" **[lesson: a known-failing test shipped to prod]**
- [ ] Coverage thresholds enforced in config (a stale coverage folder from one module is not a baseline)
- [ ] Linting + formatting (ESLint/Prettier or Biome) and ideally typechecking — zero static analysis means every typo ships
- [ ] Manual-test reference doc for the flows automation can't cover (payments UI, video/export, cross-browser)
- [ ] Cross-browser/device smoke before launch: Safari macOS, Firefox, iOS Safari + Android Chrome via tunnel on real devices
- [ ] Throttled-network test for any heavy media path (DevTools Slow 4G / Fast 3G / offline mid-operation)
- [ ] Free/paid tier limits manually verified per tier with the seeded accounts (caps trigger toasts, no sneaky API calls past the guard)

## 8. Environments

- [ ] **Separate dev/staging data from prod** — local dev must not point at the production database **[lesson: audited app's local dev wrote to the prod database]**
- [ ] Dev/prod config matrix documented: env flag, keys (test vs live), debug features, what's enforced where
- [ ] Webhook signature enforced in dev too (use `stripe listen`), so dev doesn't drift from prod behavior
- [ ] Tunnel workflow (ngrok/cloudflared) documented for real-device testing, including known interstitial/worker gotchas

## 9. Observability

- [ ] `/api/health` endpoint that actively probes critical dependencies (cached ~60s); frontend polls it and shows a maintenance notice
- [ ] Server-side error tracker: every route wrapped, sinks to storage + **auto-files deduplicated GitHub issues** (7-day dedup window, noisy-pattern allowlist)
- [ ] Infra watchdog on a cron (every 15 min): containers/services, disk, memory, swap, load, SSL expiry, OOM kills — **state-diff based** so it alerts on transitions, not every run
- [ ] Alert channel redundancy: email + a second path (ntfy/push) — and know the watchdog's blind spot if it runs on the box it monitors
- [ ] Alerts go to more than one human if more than one human operates it (bus factor)
- [ ] Daily digest email: growth, revenue, active users by tier, errors, server health — automated, archived, with a fallback if rendering fails
- [ ] Request logs shipped somewhere queryable (Logpush → R2, or equivalent)
- [ ] Product analytics consent-gated (Consent Mode v2, default-deny, IP anonymization); exclude the operator's own region from reports
- [ ] Anonymous product telemetry pipeline for key feature usage (event endpoint → DB → shows up in the daily report)

## 10. Deploy, rollback & DR

- [ ] **Enumerate every deploy surface** and document that shipping one does not ship the others (worker vs box vs scripts) — a stale copy on one surface makes new code silently no-op **[lesson]**
- [ ] One-command deploy per surface; aspire to CI-driven deploy for all of them
- [ ] Rollback runbook: exact commands, and **read the rollback target live** (`wrangler deployments list`) — never trust an ID written in a doc **[lesson]**
- [ ] Know what rollback does NOT revert (secrets, DB migrations, other deploy surfaces)
- [ ] Incident-response playbook: 30-second triage curl block (every critical endpoint + expected status), symptom → subsystem table, numbered common failure modes with 60-second fixes
- [ ] Daily automated backups of irreplaceable state (configs, env files, credentials) to off-box storage, with a freshness marker surfaced in the daily report — **verify the backup job actually runs; a dead backup script looks identical to a working one until you need it** [lesson]
- [ ] Restore runbook written and sanity-tested
- [ ] Single-point-of-failure inventory: for each SPOF, know the user-facing failure mode and the graceful degradation (e.g. planner down → clear error, not a hang)
- [ ] Server patching routine: apply security updates, reboot window, post-reboot verification command
- [ ] SSH hardening on any box: key-only auth, fail2ban, root login prohibit-password, firewalled Docker ports
- [ ] Cron TZ footgun: vixie-cron ignores `CRON_TZ` — set the system timezone deliberately **[lesson]**

## 11. Legal & compliance

- [ ] License decided (incl. dual-license/commercial terms if relevant) and declared in `package.json`
- [ ] `/terms`, `/privacy`, `/cookies` pages routed and linked from the app footer and pricing page
- [ ] Terms include the commercial-use definition and contact path if you sell personal-use licenses
- [ ] Cookie consent before any analytics loads; default-deny
- [ ] GDPR: in-app account deletion + data export (see §3)
- [ ] Transactional email deliverability plan: Supabase built-in SMTP has low limits — budget for a real ESP (Postmark/Resend/SES) before scale; don't run ops email on a personal Gmail App Password forever

## 12. Launch day (ordered)

- [ ] Full test suite green (actually green — zero failing), clean build
- [ ] Secrets pushed and counted; verify with the secret-list command
- [ ] Provider dashboards configured: Stripe (live products/prices/webhook), Supabase (Site URL, redirects, templates, OAuth), CDN/WAF (rate-limit rule — verify it live with parallel curls)
- [ ] Deploy, then post-deploy verification **on production**: signup → confirm email → OAuth → subscribe (test keys first, then live with a real card) → DB rows correct → cancel → refund-downgrade
- [ ] Stress check: heaviest realistic flow (multi-stop route, share link in incognito), health script green, no new error spikes in logs
- [ ] Error tracker verified live: throw a console error on prod → issue auto-files → delete it
- [ ] File post-launch tickets for every known gap you're consciously shipping — track, don't fix, and mark which are launch-blocking vs not
- [ ] Watch rate-limit false positives and error dashboards daily for the first week

## 13. Post-launch hygiene

- [ ] Delete legacy auth/compat paths after their grace window (e.g. old JWT secret 24h after key rotation)
- [ ] Keep the launch checklist itself updated with dates + status — it becomes your audit trail and the template for the next project
- [ ] Periodic security re-audit with findings either fixed or filed as tickets (explicitly non-blocking ones stay tickets)
- [ ] Sanity-check reporting numbers against direct DB queries once (test accounts inflate counts)
- [ ] Docs to maintain as living documents: environment.md, monitoring.md, incident-response.md, rollback.md, restore-runbook.md, security-audit.md, account-tiers.md
