# Contributing

Thanks for helping grow this checklist. The most useful contribution is a **stack-specific checklist** — a file that adds the pre-deploy checks unique to one language or ecosystem (Go, Java, Python, Rust, .NET, a framework, etc.).

## Ground rules

1. **Extend, never replace.** The base checklist ([checklists/base.md](./checklists/base.md)) already covers everything universal (auth, secrets, TLS, OWASP, backups…). A stack file adds only what is *specific* to that stack. Do not restate base items.
2. **Reuse the severity labels.** `[BLOCKER]` / `[SHOULD]` / `[NICE]`, defined in the base legend. Don't invent new ones.
3. **Every item is checkable.** Use `- [ ]` and phrase it so a person or an agent can verify it against real code/config, not opinion.
4. **Language-neutral phrasing where possible.** Name a tool only as an example, not a hard requirement.
5. **Cite authority.** If an item comes from an official security guide or style guide, link it in the file's References section.

## How to add a stack checklist

1. Copy [checklists/_template.md](./checklists/_template.md) to `checklists/<stack>.md` (lowercase, e.g. `go.md`, `java.md`, `python.md`).
2. Fill in the sections. Delete any section that has no stack-specific items.
3. Update the layout list in [README.md](./README.md) to mention your file.
4. Open a pull request. In the PR description, say which official sources you drew from.

## What makes a good stack item

Good (specific, verifiable, stack-only):
- `[BLOCKER]` Go: `govulncheck ./...` run and clean (stdlib + module CVEs)
- `[SHOULD]` Java: dependencies scanned with OWASP Dependency-Check; no known-vuln jars shipped
- `[BLOCKER]` Node: no `npm install` scripts from untrusted deps run in CI without review

Avoid (already in base, or not verifiable):
- "Use HTTPS" — already in base.
- "Write good code" — not checkable.
- "Consider security" — not an item.

## Naming

- File: `checklists/<stack>.md`, lowercase, one stack per file.
- Title (H1): `Pre-Deployment Checklist — <Stack> (extends base)`.
- Keep it short. A stack file is usually 5–20 items, not a rewrite of the base.

## Review bar

A maintainer will check that your items are (a) not duplicates of the base, (b) verifiable, (c) correctly severity-tagged, and (d) sourced where they make a security claim.
