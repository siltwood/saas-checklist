# Cloudflare Setup Checklist

DNS, TLS, WAF, Workers, R2, Access/Tunnels, logs. Items marked **[lesson]** come from the
running this stack (Workers + R2 + Access + Tunnels) in production.

## 1. Domain & DNS

- [ ] Domain on Cloudflare nameservers (or Cloudflare Registrar); registrar lock enabled
- [ ] DNSSEC enabled
- [ ] DNS records inventoried: apex, `www`, API/service subdomains, email records (MX/SPF/DKIM/DMARC — see EMAIL-LOCKDOWN)
- [ ] Proxied (orange-cloud) everything user-facing; grey-cloud only what must be direct
- [ ] `www` → apex redirect (or vice versa) — pick one canonical host
- [ ] No orphan DNS records pointing at dead infrastructure (subdomain takeover risk)

## 2. TLS/SSL

- [ ] SSL mode **Full (strict)** — "Flexible" is a silent MITM footgun
- [ ] Always Use HTTPS on; HSTS enabled once you're sure (start without preload)
- [ ] Minimum TLS version 1.2
- [ ] Certificate transparency monitoring alerts on

## 3. WAF, rate limiting & bots

- [ ] Rate-limiting rule on `/api/*` — e.g. 50 req/10s per IP, block 10s (free plan supports 10s windows) **[lesson]**
- [ ] **Verify the rule live**: fire ~100 parallel curls, confirm a mix of 200s and 429s **[lesson: verified 57/43]**
- [ ] Threshold sanity-checked against your own app's burst behavior (a single page load can fire ~19 API calls — leave headroom) **[lesson]**
- [ ] Watch for false positives the first week post-launch **[lesson]**
- [ ] Managed WAF rules on (free tier gets basics); Bot Fight Mode evaluated (can break legit API clients — test)
- [ ] Turnstile on public forms (signup, contact) instead of reCAPTCHA if you want to stay in one vendor
- [ ] Firewall rules: block obvious junk (old attack paths like `/wp-admin`, `/xmlrpc.php`) if noise bothers your logs

## 4. Workers (if API/app runs on Workers)

- [ ] Secrets via `wrangler secret put` only — never in `wrangler.toml`; verify the full set with `wrangler secret list` and count them **[lesson: "should see all 11"]**
- [ ] Custom domain routes bound (not just `*.workers.dev`); decide whether to disable the `workers.dev` URL
- [ ] `wrangler deployments list` + `wrangler rollback` drill done once — know that rollback swaps code only, not secrets, and does NOT revert other deploy surfaces **[lesson]**
- [ ] Observability/invocation logs enabled in `wrangler.toml`; Logpush → R2 if you want queryable history **[lesson]**
- [ ] Cron triggers for periodic jobs (e.g. prune expired share links) **[lesson]**
- [ ] Know your plan limits: CPU ms, subrequest count, KV read/write pricing — before launch, not after the bill
- [ ] KV is eventually consistent — fine for rate-limit counters and caches, not for anything that must be exact **[lesson]**
- [ ] Static assets: mind the per-file size cap (25 MiB) — big binaries go to R2, use `.assetsignore` to keep them out of the assets upload **[lesson]**

## 5. R2 / storage

- [ ] Buckets private by default; public access only via a Worker or custom domain with cache rules — note `r2.dev` URLs may be blocked by your own CSP **[lesson]**
- [ ] Lifecycle rules for logs/errors buckets (don't keep every error JSON forever)
- [ ] Backup story for irreplaceable bucket data (R2 has no versioning by default)

## 6. Zero Trust / Access (for internal services)

- [ ] Internal origins (self-hosted boxes) fronted by Cloudflare Tunnel — no public ports open
- [ ] Access service tokens for machine-to-machine (Worker → box); humans get SSO policies
- [ ] **Service token ID + secret are a matched pair** — on rotation, update both halves everywhere from the same rotation event **[lesson: mismatched rotation caused prod 502s]**
- [ ] Health-check path through Access documented (bare curl 403s; runbooks must include the token headers) **[lesson]**
- [ ] Tunnel process monitored (it's a SPOF for everything behind it) **[lesson: health-watch checks cloudflared]**

## 7. Caching & pages

- [ ] Cache rules: static assets long-TTL, API paths bypass
- [ ] Purge-cache step in the deploy runbook if you serve versioned assets without hashes
- [ ] Custom error pages if you care (default Cloudflare 5xx page is ugly and off-brand)

## 8. Account security

- [ ] 2FA (hardware key/passkey) on the Cloudflare account; no shared logins — use account members with scoped roles
- [ ] API tokens scoped least-privilege per use (analytics-read for reports ≠ zone-edit) — **never commit them** **[lesson: a CF token was once hardcoded in a script and had to be rotated]**
- [ ] Notification emails on: usage spikes, security events, SSL expiry, Worker error rates
