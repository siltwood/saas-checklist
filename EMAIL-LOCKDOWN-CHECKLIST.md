# Email & Gmail Lockdown Checklist

Two halves: (A) securing the Google/Gmail accounts that own your infrastructure, and
(B) making your domain's email deliverable and unspoofable. Items marked **[lesson]**
come from a real production launch.

## A. Lock down the Google account(s)

The Gmail account that owns your GCP project, Play/Workspace, domains, and recovery flows
is the master key to the whole business. Treat it accordingly.

- [ ] Inventory which Google account owns what: GCP project, Analytics, Search Console, OAuth app, YouTube/socials recovery — consolidate deliberately, not accidentally
- [ ] **2FA with passkeys or hardware security keys** — remove SMS as a second factor (SIM-swap risk); keep printed backup codes somewhere physical
- [ ] Consider Google Advanced Protection Program for the account that owns production
- [ ] Recovery email + phone reviewed: they must point at things you control, not an old address
- [ ] Security checkup run (https://myaccount.google.com/security): review signed-in devices, third-party app access, revoke everything stale
- [ ] **App Passwords audited** (https://myaccount.google.com/apppasswords): one per consumer, named clearly, revoke unused ones
- [ ] App Passwords are secrets: never in `.dev.vars`/repo files, never pasted in chat — **rotate immediately on any leak** (revoke old → generate new → update the one consumer → smoke-test) **[lesson: leaked App Password had to be re-rotated after landing in a dotfile + chat]**
- [ ] Separate concerns: personal Gmail ≠ ops account. If ops email rides a personal Gmail (fine at small scale), write down the migration path to a real address/ESP before it calcifies **[lesson]**
- [ ] Delegate/emergency access plan: if you're solo, a trusted person (or documented recovery kit) can get in if you're unavailable — bus factor applies to accounts too **[lesson: single operator is sole alert recipient]**

## B. Domain email: deliverability + anti-spoofing

Do this even if you "don't send email" — without DMARC anyone can spoof your domain.

- [ ] Decide sending architecture: auth emails (Supabase/auth provider SMTP), transactional (ESP: Postmark/Resend/SES), ops/alerts (Gmail SMTP is fine early) — know which system sends what **[lesson: three separate senders, each with its own sender-name setting]**
- [ ] **SPF** record: one TXT record including every legitimate sender (`include:` per ESP), ending `~all` → `-all` once verified. Max 10 DNS lookups — flatten if needed
- [ ] **DKIM** enabled and verified per sender (each ESP + Google gets its own selector)
- [ ] **DMARC**: start `p=none` with `rua=` reports, watch a week, then move to `p=quarantine` → `p=reject`
- [ ] MX records correct; inbound mail for `support@`/`noreply@` routed somewhere monitored (Cloudflare Email Routing is a free way to forward)
- [ ] Sender display name and from-address consistent across every system that sends ("Product" <noreply@domain.com>) — check each provider's separate setting **[lesson: sender name mismatched between Gmail SMTP and Supabase auth emails]**
- [ ] Test deliverability: send to Gmail/Outlook/iCloud, check spam placement + headers (mail-tester.com or check-auth headers manually)
- [ ] Auth email templates customized and tested end-to-end (confirmation link actually lands the user signed in, on production)
- [ ] Know your provider's sending limits (Supabase built-in SMTP is very low; Gmail ~500/day) and the upgrade trigger before you hit it **[lesson]**
- [ ] Unsubscribe/one-click headers on anything marketing-flavored (see GDPR checklist §6)
