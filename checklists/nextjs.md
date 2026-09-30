# Pre-Deployment Checklist — Next.js (extends base)

Stack-specific pre-deploy checks for **Next.js** (React, App Router or Pages Router). Run [base.md](./base.md) first; these items are *additional*. Severity labels (`[BLOCKER]` / `[SHOULD]` / `[NICE]`) are defined in the base legend.

Items are stated by intent; the file / config key / command in parentheses is the usual way to satisfy them.

## Dependencies & build
- [ ] `[BLOCKER]` Production build succeeds with no type errors and no ESLint errors (`next build`) — build not passing with `ignoreBuildErrors` / `ignoreDuringBuilds` set to hide failures
- [ ] `[BLOCKER]` Dependency audit clean (`npm audit` / `pnpm audit`), or findings triaged; lockfile committed
- [ ] `[SHOULD]` Node version pinned (`engines` in `package.json` / `.nvmrc`) and matches the deploy target
- [ ] `[SHOULD]` Bundle analyzed for unexpected bloat / duplicate deps (`@next/bundle-analyzer`)

## Environment & secrets
- [ ] `[BLOCKER]` Only truly public values use the `NEXT_PUBLIC_` prefix — anything `NEXT_PUBLIC_` is inlined into the client bundle and shipped to the browser (never put API secrets, DB URLs, or private keys behind it)
- [ ] `[BLOCKER]` Server-only secrets read from non-`NEXT_PUBLIC_` env vars and used only in server code (Route Handlers, Server Components, Server Actions, `getServerSideProps`)
- [ ] `[BLOCKER]` `.env*` files git-ignored; env vars set per-environment in the host (Vercel/host dashboard), not committed
- [ ] `[SHOULD]` Required env vars validated at startup/build (e.g. a `zod`-parsed env module) so a missing var fails loud, not at runtime

## Rendering, caching & data
- [ ] `[BLOCKER]` Dynamic/user-specific pages are not accidentally static-cached — routes reading cookies/headers/auth use dynamic rendering (`export const dynamic = 'force-dynamic'` or `cookies()`/`headers()` usage) so one user's data isn't served to another
- [ ] `[BLOCKER]` `fetch()` cache semantics deliberate — `cache: 'no-store'` for per-user/authorized data, revalidation set intentionally for shared data (no stale private data in the Full Route Cache / Data Cache)
- [ ] `[SHOULD]` ISR / `revalidate` values chosen per route; on-demand revalidation (`revalidatePath`/`revalidateTag`) wired where content changes
- [ ] `[SHOULD]` `generateStaticParams` covers the intended paths; fallback behavior for unknown params defined
- [ ] `[SHOULD]` `<Image>` used with correct `remotePatterns` in `next.config` (no wildcard `**` host allowing arbitrary remote image proxying)

## API routes, Server Actions & middleware
- [ ] `[BLOCKER]` Every Route Handler / Server Action authenticates AND authorizes server-side — Server Actions are public POST endpoints; do not assume they're only callable from your UI
- [ ] `[BLOCKER]` Input validated server-side in handlers/actions (e.g. `zod`), including IDs (prevent IDOR — check the object belongs to the caller)
- [ ] `[BLOCKER]` `middleware.ts` is not the only auth gate for sensitive data — enforce authz again in the handler/component (middleware can be bypassed for some request types and doesn't run on all paths)
- [ ] `[SHOULD]` SSRF guard on any server-side `fetch` of a user-supplied URL (block internal ranges / metadata endpoints)
- [ ] `[SHOULD]` Rate limiting on auth + expensive routes (e.g. Upstash ratelimit) using a trusted client IP source behind the CDN

## Security headers & client
- [ ] `[BLOCKER]` Security headers set (in `next.config` `headers()` or middleware): `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`, `Referrer-Policy`, `X-Frame-Options`/`frame-ancestors`
- [ ] `[BLOCKER]` No server secrets referenced in Client Components (`'use client'`) — a leaked secret in client code ships to every browser
- [ ] `[SHOULD]` `dangerouslySetInnerHTML` only on sanitized/trusted HTML (XSS)
- [ ] `[SHOULD]` Source maps not exposing internal source to the public in prod (or access-restricted); `poweredByHeader: false`
- [ ] `[SHOULD]` `redirect()` / `next/link` targets validated when derived from user input (open-redirect)

## Runtime, deploy & observability
- [ ] `[BLOCKER]` Runtime chosen deliberately per route (Node vs Edge) — code using Node-only APIs (fs, some SDKs) not deployed to the Edge runtime; long/heavy work not forced onto Edge limits
- [ ] `[BLOCKER]` `next start` serves the production build (not `next dev`); or the host's production output (standalone / adapter) is what's deployed
- [ ] `[SHOULD]` `images`, `redirects`, `rewrites`, and `basePath` in `next.config` verified against prod domain; trailing-slash/canonical behavior consistent
- [ ] `[SHOULD]` Error boundaries + `error.tsx` / `not-found.tsx` present so failures show a clean page, not a raw stack trace
- [ ] `[SHOULD]` Error tracking + analytics wired (e.g. Sentry, Vercel Analytics/Speed Insights) with no PII/secrets in logs
- [ ] `[NICE]` `robots`, `sitemap`, and metadata (`generateMetadata`) correct for the production domain

## References
- Next.js Deploying: https://nextjs.org/docs/app/building-your-application/deploying
- Production checklist (official): https://nextjs.org/docs/app/building-your-application/deploying/production-checklist
- Environment variables: https://nextjs.org/docs/app/building-your-application/configuring/environment-variables
- Caching & revalidating: https://nextjs.org/docs/app/building-your-application/caching
- Data Security in Next.js: https://nextjs.org/blog/security-nextjs-server-components-actions
- next.config headers: https://nextjs.org/docs/app/api-reference/config/next-config-js/headers
