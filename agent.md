# Pre-Deployment Checklist — AI Agent Version

Machine-facing companion to [checklist.md](./checklist.md). The human file is the **checklist** (the source of truth for *what* to verify). This file tells an AI coding agent (Claude Code, Cursor, Copilot, etc.) *how* to run it before a deploy.

Point your agent at both files. The checklist provides the items; this file provides the rules.

---

## 1. Copy-paste prompt

Paste this into your agent, in the repo you are about to deploy:

```text
Read docs/deployment-checklist/checklist.md and docs/deployment-checklist/agent.md.

Audit THIS repository against every checklist item that applies to it:
- Web-only repo: skip the Mobile-Specific section.
- Mobile-only repo: skip the Web-Specific section.
- Full-stack / both: check everything.

For EACH item:
- Verify against the actual code, config, and dependencies. Do NOT assume.
- Cite the evidence: a file path + line, a config value, or command output.
- Mark: PASS / FAIL / N/A. N/A must include a one-line reason.

Then output, in this order:
1. Summary counts: PASS / FAIL / N/A totals.
2. A FAIL table: item, section, severity ([BLOCKER]/[SHOULD]/[NICE]), evidence, exact fix.
3. Verdict: GO or NO-GO. NO-GO if ANY [BLOCKER] is FAIL.

Rules:
- Report only. Do NOT deploy. Do NOT run destructive commands.
- Do NOT fix anything yet — list the fixes and wait for my approval.
```

---

## 2. Output contract

The agent's report must contain these three parts, in order.

### 2.1 Summary counts

```text
PASS: <n>   FAIL: <n>   N/A: <n>   (total <n> items checked)
```

### 2.2 FAIL table

| Item | Section | Severity | Evidence | Fix |
|------|---------|----------|----------|-----|
| (short item text) | (e.g. Web / Security 4.2) | `[BLOCKER]` | (file:line or config or command output) | (exact change) |

Only FAIL rows appear here. If there are zero FAILs, write `No failures.`

### 2.3 Verdict

```text
VERDICT: GO
```
or
```text
VERDICT: NO-GO — <n> blocker(s) unresolved
```

**Rule: NO-GO if any `[BLOCKER]` item is FAIL.** A single unresolved blocker blocks the whole deploy, regardless of how many other items pass.

---

## 3. Verify-not-assume rule

This is the rule that makes the audit trustworthy.

- **Evidence or it's a FAIL.** An item is PASS only with proof: a file path + line, a concrete config value, or the output of a command the agent ran. "Looks fine", "probably set", or "the framework handles this" is **not** evidence — mark it FAIL or investigate further.
- **Check the real artifact, not the intent.** For "no secrets in repo", the agent greps the history, not just the current tree. For "TLS only", it inspects the actual config, not a comment claiming it.
- **`N/A` must be justified.** Example: `PCI DSS — N/A: no card data handled, payments delegated to Stripe Checkout (see app/checkout/route.ts).`
- **Report first, fix second.** The agent reports findings and proposed fixes. A human approves before any fix is applied and before any deploy. The agent must never deploy or run destructive commands on its own.

---

## 4. Notes

- Keep this file and [checklist.md](./checklist.md) in sync. When you add a checklist item, the agent picks it up automatically — no change needed here unless the output format changes.
- Severity labels (`[BLOCKER]` / `[SHOULD]` / `[NICE]`) are defined in the human checklist's legend.
- The human **Sign-off** block still applies: a person records the final GO/NO-GO, even when an agent produced the audit.
