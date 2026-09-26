# Pre-Deployment Checklist — <Stack> (extends base)

Stack-specific pre-deploy checks for **<Stack>**. Run [base.md](./base.md) first; these items are *additional*. Severity labels (`[BLOCKER]` / `[SHOULD]` / `[NICE]`) are defined in the base legend.

> Delete this line and any empty section before submitting. Keep only items unique to <Stack> — do not restate base items.

## Dependencies & supply chain
- [ ] `[BLOCKER]` (e.g. run the stack's vulnerability scanner and confirm clean)
- [ ] `[SHOULD]` (e.g. lockfile / manifest pinned; no floating versions)

## Build & release
- [ ] `[BLOCKER]` (e.g. release build flags strip debug symbols / disable dev mode)
- [ ] `[SHOULD]` (e.g. reproducible build verified with the stack's tooling)

## Runtime & config
- [ ] `[BLOCKER]` (e.g. framework debug/dev mode is OFF in production config)
- [ ] `[SHOULD]` (e.g. safe defaults for the stack's server/runtime are set)

## Security specifics
- [ ] `[BLOCKER]` (e.g. deserialization / template / query APIs used safely for this stack)
- [ ] `[SHOULD]` (e.g. stack-specific linter security rules enabled in CI)

## References
- (Link the official security guide / hardening doc for this stack.)
