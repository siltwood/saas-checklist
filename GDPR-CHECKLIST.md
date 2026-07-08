# GDPR / Privacy Readiness Checklist

For any product with EU/UK users (i.e., any public web product). Grounded in what a small SaaS
actually needs — not enterprise-DPO theater. Items marked **[lesson]** come from a real production launch.

## 1. Data inventory (do this first — everything else depends on it)

- [ ] List every category of personal data you collect: account email, OAuth profile, IP addresses, payment data, user content, analytics identifiers, error logs, support emails
- [ ] For each: where it's stored (DB, R2/S3, logs, Stripe, GA4, email provider), why you need it (purpose), and how long you keep it (retention)
- [ ] List every subprocessor that touches personal data (e.g. Supabase, Stripe, Cloudflare, Google Analytics, email provider, error tracker) — this becomes your privacy policy's third-party section
- [ ] Check where each subprocessor stores data (region) and whether they offer a DPA — sign/accept the DPA for each (usually a dashboard checkbox: Stripe, Supabase, Google all have standard ones)

## 2. Lawful basis & consent

- [ ] Map each processing purpose to a lawful basis: contract (account, billing), legitimate interest (security logs, fraud), consent (analytics, marketing)
- [ ] Analytics/marketing cookies load **only after consent** — default-deny (Google Consent Mode v2 with all signals denied until the user accepts)
- [ ] Consent is granular where needed (analytics vs marketing), freely declinable, and the decline path is as easy as accept
- [ ] Consent choice is persisted and revocable (cookie settings page or re-openable banner)
- [ ] No dark patterns: no pre-ticked boxes, no "accept" 5× bigger than "decline"
- [ ] IP anonymization on in analytics; Google Signals / ad personalization off unless you truly need it

## 3. Privacy policy & cookie policy

- [ ] `/privacy` page routed and linked from footer, signup, and cookie banner
- [ ] Policy covers: what you collect, why, lawful basis, retention, subprocessors (named), international transfers, user rights, contact address, last-updated date
- [ ] `/cookies` page listing each cookie/localStorage key, purpose, and lifetime
- [ ] Written in plain language — a policy nobody can read doesn't count
- [ ] Policy updated whenever you add a subprocessor or new data category (tie it to your launch checklist)

## 4. User rights (the part that needs code)

- [ ] **Right to erasure**: in-app account deletion that actually cascades — auth user, DB rows, and a plan for backups/logs. Server-side RPC, not a "email us" form **[lesson: `delete_own_account()` RPC]**
- [ ] **Right to access/portability**: in-app data export (JSON) of everything tied to the account **[lesson: `export_own_account()` RPC]**
- [ ] Deletions/exports are logged as account events (proof of compliance, debugging)
- [ ] Stripe customer handling on delete: you can keep invoices (legal obligation) but document that in the policy
- [ ] Error tracker / logs redact PII before storage (emails, UUIDs, tokens) — logs count as personal data **[lesson: `redactPII` in the error tracker]**
- [ ] Test the full delete path: create account → add data → subscribe (test mode) → delete → verify nothing user-identifiable remains queryable

## 5. Security obligations

- [ ] Encryption in transit everywhere (TLS, HSTS); at rest via your providers (verify, don't assume)
- [ ] Access control: RLS/row-level isolation so users can only reach their own data
- [ ] Breach plan: know your 72-hour notification obligation; incident-response doc includes "does this incident expose personal data?" as a triage question
- [ ] Data minimization: don't collect what you don't use (do you really need the IP in that table?)
- [ ] Retention enforced, not just stated: cron/lifecycle rules that actually delete old logs, stale share links, expired sessions

## 6. Email & marketing

- [ ] Transactional email (receipts, resets) needs no consent; marketing email does — separate the lists
- [ ] Every marketing email has a working one-click unsubscribe
- [ ] Don't auto-enroll signups into a newsletter without an explicit checkbox (unticked)

## 7. Paperwork (lightweight version for a small SaaS)

- [ ] Records of processing: your data inventory from §1, kept in the repo as a living doc — that IS the record
- [ ] DPAs accepted with all subprocessors (screenshot/date them)
- [ ] If you sell B2B: have a DPA template ready for customers who ask
- [ ] EU representative / DPO: almost certainly not required at small scale — note the thresholds and revisit if you grow
