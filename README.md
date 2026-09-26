# Deployment Checklist

A reusable, language- and framework-agnostic checklist to run before **every** production deploy. Covers web apps and mobile apps, with a deep, audit-grade security section (threat model, OWASP Web + Mobile Top 10, secrets, compliance, incident response).

Works for any stack. Copy it into your own repo and run it per release.

## Two ways to use it

| File | For | What it is |
|------|-----|------------|
| [checklist.md](./checklist.md) | **Humans** | The checklist itself — tick the `- [ ]` boxes, sign off, deploy. Source of truth for *what* to verify. |
| [agent.md](./agent.md) | **AI agents** | A copy-paste prompt + output contract so a coding agent (Claude Code, Cursor, Copilot, etc.) audits your repo against the checklist and gives a GO / NO-GO verdict. |

## Quick start

**By hand:** open [checklist.md](./checklist.md), copy it per release, tick each box. Any unchecked `[BLOCKER]` = no deploy.

**With an AI agent:** open [agent.md](./agent.md), copy the prompt into your agent inside the repo you're deploying. It audits, cites evidence, and returns GO / NO-GO.

## Severity

| Mark | Meaning |
|------|---------|
| `[BLOCKER]` | Do not deploy until fixed |
| `[SHOULD]` | Deploy only with a tracked ticket + owner |
| `[NICE]` | Improve when time allows |

## Sections in the checklist

1. Shared / Universal (build, secrets, auth, data, observability, testing, dependencies)
2. Web-Specific (headers, TLS, CORS, XSS/CSRF/SSRF, abuse protection, client bundle)
3. Mobile-Specific (signing, permissions, secure storage, network, anti-tampering)
4. Security — Audit-Grade (threat model, OWASP Web + Mobile Top 10, secrets/KMS, pentest, compliance, incident response)
5. Sign-off
6. References

## License / reuse

Copy, fork, adapt. Keep [checklist.md](./checklist.md) as the source of truth and let [agent.md](./agent.md) point at it, so both stay in sync.
