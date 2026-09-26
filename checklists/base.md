# Pre-Deployment Checklist — Base (Web & Mobile)

Language-agnostic, framework-agnostic. Run this before **every** production deploy. Security is the deepest section on purpose — do not skip it.

> This is the **base** checklist — it applies to any stack. Stack-specific checklists (e.g. [`go.md`](./go.md), [`java.md`](./java.md)) *extend* this file with extra items; they never replace it. Run the base first, then any stack file that matches your project.

## How to use (humans)

1. Copy this file (or the relevant sections) per release.
2. Tick each `- [ ]` box. A `[BLOCKER]` left unchecked = **no deploy**.
3. Fill the **Sign-off** block at the end. Get a second person to approve when possible.

## How to use (AI agents)

Running this with an AI coding agent (Claude Code, Cursor, Copilot, etc.)? See [agent.md](../agent.md) — it has a copy-paste prompt, the PASS/FAIL/N/A output contract, and the verify-not-assume rule. This file stays the source of truth for *what* to check.

**Severity legend**

| Mark | Meaning |
|------|---------|
| `[BLOCKER]` | Do not deploy until fixed |
| `[SHOULD]` | Deploy only with a tracked ticket + owner |
| `[NICE]` | Improve when time allows |

---

## 1. Shared / Universal (Web + Mobile)

### Build & release
- [ ] `[BLOCKER]` Build is reproducible from a clean checkout (no local-only files, no uncommitted state)
- [ ] `[BLOCKER]` Version / build number bumped and tagged in version control
- [ ] `[SHOULD]` Changelog / release notes updated
- [ ] `[BLOCKER]` Rollback plan written and tested (know how to revert in < 5 min)
- [ ] `[SHOULD]` Feature flags for new work default to **off**; toggles documented
- [ ] `[NICE]` Build artifacts are stored/archived for the release tag

### Config & secrets
- [ ] `[BLOCKER]` No secrets, API keys, tokens, or credentials committed to the repo (check history, not just HEAD)
- [ ] `[BLOCKER]` Environment-specific config separated (dev / staging / prod); prod config not reachable from dev
- [ ] `[BLOCKER]` `.gitignore` (or equivalent) covers key files, `.env`, certificates, keystores
- [ ] `[SHOULD]` Secret rotation policy exists; long-lived secrets flagged for rotation
- [ ] `[SHOULD]` Third-party keys are scoped to least privilege (read-only where possible)

### Auth & sessions
- [ ] `[BLOCKER]` Session / access tokens have a sane expiry
- [ ] `[BLOCKER]` Refresh flow works and revokes old tokens
- [ ] `[BLOCKER]` Logout invalidates the session server-side (not just client-side)
- [ ] `[SHOULD]` Password policy + login rate-limit / lockout in place
- [ ] `[SHOULD]` Account recovery flow reviewed (no user enumeration, no insecure reset links)

### Data & privacy
- [ ] `[BLOCKER]` All data encrypted in transit (TLS everywhere)
- [ ] `[BLOCKER]` Sensitive data encrypted at rest
- [ ] `[SHOULD]` PII minimized — collect only what is needed
- [ ] `[BLOCKER]` Backups exist **and a restore has been tested** (a backup you can't restore is not a backup)
- [ ] `[SHOULD]` Data deletion path works (user delete removes real data, not just a flag)

### Observability
- [ ] `[BLOCKER]` Logs contain **no** secrets, tokens, or PII
- [ ] `[SHOULD]` Error tracking wired up (crash / exception reporting)
- [ ] `[SHOULD]` Alerting on error-rate spikes and downtime
- [ ] `[SHOULD]` Health check / readiness endpoint or heartbeat exists
- [ ] `[NICE]` Dashboards for key metrics (latency, error rate, traffic)

### Testing & QA
- [ ] `[BLOCKER]` Unit + integration tests pass in CI
- [ ] `[SHOULD]` End-to-end / critical-path tests pass
- [ ] `[BLOCKER]` Smoke test on staging with production-like config
- [ ] `[SHOULD]` Load / stress test for expected peak traffic
- [ ] `[NICE]` Manual exploratory pass on the main user flows

### Dependencies
- [ ] `[BLOCKER]` Lockfile committed (deps pinned to exact/resolved versions)
- [ ] `[BLOCKER]` Dependency vulnerability scan clean (or known issues triaged + tracked)
- [ ] `[SHOULD]` No abandoned / unmaintained critical dependencies
- [ ] `[SHOULD]` License compliance checked for shipped dependencies

---

## 2. Web-Specific

### Security headers
- [ ] `[BLOCKER]` `Content-Security-Policy` set and restrictive (no blanket `unsafe-inline` on scripts)
- [ ] `[BLOCKER]` `Strict-Transport-Security` (HSTS) enabled
- [ ] `[SHOULD]` `X-Frame-Options` / `frame-ancestors` set (clickjacking)
- [ ] `[SHOULD]` `X-Content-Type-Options: nosniff`
- [ ] `[SHOULD]` `Referrer-Policy` set
- [ ] `[NICE]` `Permissions-Policy` restricts camera / mic / geolocation etc.

### Transport
- [ ] `[BLOCKER]` Valid TLS certificate; not expiring soon; auto-renew confirmed
- [ ] `[BLOCKER]` All `http://` redirects to `https://`
- [ ] `[NICE]` HSTS preload considered for the apex domain

### Cross-origin & cookies
- [ ] `[BLOCKER]` CORS locked to known origins (no `*` on authenticated endpoints)
- [ ] `[BLOCKER]` Session cookies are `HttpOnly` + `Secure` + `SameSite`

### Input / output
- [ ] `[BLOCKER]` All input validated server-side (client validation is UX only, not security)
- [ ] `[BLOCKER]` Output encoded / escaped to prevent XSS
- [ ] `[BLOCKER]` CSRF protection on state-changing requests
- [ ] `[BLOCKER]` SSRF guards on any server-side URL fetch (block internal IPs / metadata endpoints)

### Abuse protection
- [ ] `[BLOCKER]` Rate limiting on auth + expensive endpoints
- [ ] `[SHOULD]` Bot protection / CAPTCHA on signup, login, contact forms where relevant
- [ ] `[SHOULD]` CDN / DDoS protection in front of origin

### Client bundle
- [ ] `[BLOCKER]` No secrets baked into client-side bundle (anything shipped to the browser is public)
- [ ] `[SHOULD]` Source maps not exposing internal source in prod (or access-restricted)
- [ ] `[BLOCKER]` Debug endpoints / admin routes not reachable in prod

### Pre-deploy hygiene (non-security)
- [ ] `[SHOULD]` `robots.txt`, sitemap, canonical URLs correct
- [ ] `[NICE]` Performance budget / Lighthouse pass
- [ ] `[SHOULD]` Accessibility (a11y) pass on key pages
- [ ] `[NICE]` Custom 404 / 500 pages don't leak stack traces

---

## 3. Mobile-Specific

> Applies to any platform or framework (native iOS/Android, cross-platform, hybrid). Items are stated by intent, not by tool. Tool names below are examples only.

### Store readiness
- [ ] `[BLOCKER]` App correctly signed (release signing keys/certs/provisioning, not debug)
- [ ] `[BLOCKER]` App ID / bundle identifier matches the store listing
- [ ] `[BLOCKER]` Version name + build number incremented
- [ ] `[SHOULD]` Store listing complete (description, screenshots, category, contact)
- [ ] `[BLOCKER]` Privacy disclosure filled (data-safety / privacy nutrition label) and matches actual data use

### Permissions
- [ ] `[BLOCKER]` Request only the permissions actually used
- [ ] `[SHOULD]` Each permission has a clear user-facing justification string
- [ ] `[BLOCKER]` Runtime permission prompts handled; app degrades gracefully when denied

### Secure storage
- [ ] `[BLOCKER]` Secrets / tokens stored in platform secure storage (Keychain / Keystore), never plaintext files or shared prefs
- [ ] `[BLOCKER]` No secrets, private keys, or backend admin creds hardcoded in the binary (the binary ships to the user's device and can be reverse-engineered)

### Network
- [ ] `[BLOCKER]` TLS only; cleartext / plain-HTTP traffic disabled
- [ ] `[SHOULD]` Certificate pinning where the threat model warrants it
- [ ] `[SHOULD]` API keys embedded in the app are treated as public (scope them; enforce auth server-side)

### Anti-tampering & release hygiene
- [ ] `[SHOULD]` Root / jailbreak awareness for sensitive flows (payments, auth)
- [ ] `[SHOULD]` Sensitive logic obfuscated / minified in release build
- [ ] `[BLOCKER]` Debug logging, verbose logs, and dev tooling disabled in the release build

### Offline & data
- [ ] `[SHOULD]` Local cache / database encrypted if it holds sensitive data
- [ ] `[BLOCKER]` Local data wiped on logout
- [ ] `[BLOCKER]` Deep links / URL schemes validated (reject untrusted params, no auth bypass via link)

### Update & recovery
- [ ] `[SHOULD]` Over-the-air / update path tested
- [ ] `[SHOULD]` Force-update mechanism for a broken/insecure old version
- [ ] `[SHOULD]` Crash reporting enabled (without leaking PII)

---

## 4. Security — Audit-Grade

This is the deep section. Treat it as its own gate.

### 4.1 Threat model quickstart
- [ ] `[BLOCKER]` **Assets** listed — what an attacker wants (user data, credentials, money, uptime, reputation)
- [ ] `[BLOCKER]` **Entry points** listed — every place input crosses into the system (APIs, forms, file upload, webhooks, deep links, third-party callbacks)
- [ ] `[SHOULD]` **Trust boundaries** drawn — where does untrusted become trusted? (client to server, service to service, tenant to tenant)
- [ ] `[SHOULD]` **STRIDE** pass — for each entry point, ask:
  - Spoofing — can identity be faked?
  - Tampering — can data be altered in transit / at rest?
  - Repudiation — can an action be denied? (is it logged?)
  - Information disclosure — can data leak?
  - Denial of service — can it be overwhelmed?
  - Elevation of privilege — can a user gain rights they shouldn't have?

### 4.2 OWASP Top 10 (Web) mapping

| # | Category | Check | Severity | Done |
|---|----------|-------|----------|------|
| A01 | Broken Access Control | Every endpoint enforces authorization server-side; no IDOR (object IDs checked against the caller) | `[BLOCKER]` | [ ] |
| A02 | Cryptographic Failures | Strong TLS; no weak/legacy ciphers; passwords hashed with a modern KDF, not encrypted/plaintext | `[BLOCKER]` | [ ] |
| A03 | Injection | Parameterized queries / ORM; no string-built SQL/NoSQL/OS commands; template injection guarded | `[BLOCKER]` | [ ] |
| A04 | Insecure Design | Abuse cases considered, not just happy path; rate limits + business-logic limits exist | `[SHOULD]` | [ ] |
| A05 | Security Misconfiguration | No default creds; error messages don't leak internals; unused features/ports disabled | `[BLOCKER]` | [ ] |
| A06 | Vulnerable & Outdated Components | SCA scan clean; components patched | `[BLOCKER]` | [ ] |
| A07 | Identification & Auth Failures | MFA available; brute-force lockout; secure session mgmt (see 4.4) | `[BLOCKER]` | [ ] |
| A08 | Software & Data Integrity Failures | CI/CD supply chain trusted; no unsigned/unverified update or deserialization of untrusted data | `[SHOULD]` | [ ] |
| A09 | Security Logging & Monitoring Failures | Security events logged (auth, access-denied, admin actions); alerts on anomalies | `[SHOULD]` | [ ] |
| A10 | Server-Side Request Forgery | Server-side fetches validate/allowlist targets; internal ranges + metadata endpoints blocked | `[BLOCKER]` | [ ] |

### 4.3 OWASP Mobile Top 10 mapping

| # | Category | Check | Severity | Done |
|---|----------|-------|----------|------|
| M1 | Improper Credential Usage | No hardcoded creds; tokens in secure storage | `[BLOCKER]` | [ ] |
| M2 | Inadequate Supply Chain Security | SDKs/deps vetted; build pipeline trusted | `[SHOULD]` | [ ] |
| M3 | Insecure Auth/Authz | Auth enforced server-side; no client-only gating | `[BLOCKER]` | [ ] |
| M4 | Insufficient Input/Output Validation | Deep links, IPC, file input validated | `[BLOCKER]` | [ ] |
| M5 | Insecure Communication | TLS only; cert validation not disabled; pinning where needed | `[BLOCKER]` | [ ] |
| M6 | Inadequate Privacy Controls | Data-safety disclosure accurate; minimal collection | `[SHOULD]` | [ ] |
| M7 | Insufficient Binary Protection | Release build stripped of debug; sensitive logic obfuscated | `[SHOULD]` | [ ] |
| M8 | Security Misconfiguration | Debuggable flag off; backup flags reviewed; exported components locked down | `[BLOCKER]` | [ ] |
| M9 | Insecure Data Storage | No sensitive data in plaintext/logs/cache | `[BLOCKER]` | [ ] |
| M10 | Insufficient Cryptography | Platform crypto APIs used; no home-rolled crypto or weak algorithms | `[SHOULD]` | [ ] |

### 4.4 AuthN / AuthZ deep
- [ ] `[BLOCKER]` **Server-side authorization on every request** — never trust the client to hide/disable actions
- [ ] `[BLOCKER]` **IDOR check** — accessing another user's/tenant's object by ID is denied
- [ ] `[SHOULD]` **Least privilege** — roles/scopes grant the minimum; admin paths separated
- [ ] `[SHOULD]` **MFA** available for privileged / high-value accounts
- [ ] `[BLOCKER]` **Brute-force protection** — lockout / backoff on auth endpoints
- [ ] `[SHOULD]` Multi-tenant isolation verified (tenant A cannot read tenant B)

### 4.5 Secrets & key management
- [ ] `[BLOCKER]` Secrets live in a vault / secret manager / KMS, not in code or plain env files in the repo
- [ ] `[BLOCKER]` No long-lived static keys where short-lived / rotating credentials are possible
- [ ] `[SHOULD]` Rotation policy defined and a rotation has actually been rehearsed
- [ ] `[SHOULD]` Each credential scoped to least privilege and to a single purpose

### 4.6 Pre-deploy pentest items
- [ ] `[SHOULD]` **SAST** (static analysis) run on the codebase; findings triaged
- [ ] `[SHOULD]` **DAST** (dynamic scan) run against staging
- [ ] `[BLOCKER]` **Dependency / SCA scan** clean or triaged
- [ ] `[BLOCKER]` **Secret scanner** run over full git history (not just current files)
- [ ] `[SHOULD]` Manual spot-checks: auth bypass, IDOR, injection on top 3 riskiest endpoints
- [ ] `[NICE]` Third-party pentest booked for major releases

### 4.7 Compliance
- [ ] `[BLOCKER]` Privacy policy published and accurate to actual data use
- [ ] `[SHOULD]` **GDPR** (if EU users): lawful basis / consent, data-subject access + deletion, Data Processing Agreements with sub-processors
- [ ] `[SHOULD]` Data retention policy defined (how long, then deleted)
- [ ] `[SHOULD]` Cookie / tracking consent banner where required
- [ ] `[BLOCKER]` Breach-notification readiness (who, when, how — regulators + users)
- [ ] `[BLOCKER]` **PCI DSS** if handling card data — prefer a compliant processor; never store raw PAN
- [ ] `[NICE]` Accessibility / regional legal requirements checked

### 4.8 Incident response
- [ ] `[BLOCKER]` On-call / security contact defined and reachable
- [ ] `[BLOCKER]` Runbook exists: detect, contain, eradicate, recover, learn
- [ ] `[BLOCKER]` Emergency procedure to **rollback** and **revoke keys** documented and tested
- [ ] `[SHOULD]` `security.txt` / disclosure channel published for external reporters
- [ ] `[NICE]` Post-mortem template ready (blameless, action items tracked)

---

## 5. Sign-off

| Field | Value |
|-------|-------|
| Release name / version | |
| Date | |
| Platform(s) | Web / iOS / Android / other |
| Checked by | |
| Approved by | |
| Open `[SHOULD]` items (with ticket links) | |
| **Go / No-Go** | |

> Rule: **zero unresolved `[BLOCKER]`.** Every `[SHOULD]` shipping open must have an owner and a ticket.

---

## 6. References

- OWASP Top 10 (Web): https://owasp.org/www-project-top-ten/
- OWASP Mobile Top 10: https://owasp.org/www-project-mobile-top-10/
- OWASP MASVS (Mobile App Security Verification Standard): https://mas.owasp.org/MASVS/
- OWASP ASVS (Application Security Verification Standard): https://owasp.org/www-project-application-security-verification-standard/
- OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
- Mozilla Observatory (headers / TLS): https://developer.mozilla.org/en-US/observatory
- Security Headers scanner: https://securityheaders.com/
- SSL Labs (TLS test): https://www.ssllabs.com/ssltest/
- GDPR checklist: https://gdpr.eu/checklist/

---

*Reusable across any language / framework. Keep this doc in version control; update the checklist as new threat classes appear.*
