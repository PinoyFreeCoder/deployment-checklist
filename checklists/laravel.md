# Pre-Deployment Checklist — Laravel (extends base)

Stack-specific pre-deploy checks for **Laravel** (PHP). Run [base.md](./base.md) first; these items are *additional*. Severity labels (`[BLOCKER]` / `[SHOULD]` / `[NICE]`) are defined in the base legend.

Items are stated by intent; the artisan command / config key in parentheses is the usual way to satisfy them.

## Dependencies & supply chain
- [ ] `[BLOCKER]` Production install excludes dev packages (`composer install --no-dev --optimize-autoloader`) — Telescope, Debugbar, Faker, Tinker helpers not shipped
- [ ] `[BLOCKER]` Dependency audit clean (`composer audit`) — no known-vuln packages
- [ ] `[SHOULD]` `composer.lock` committed; deploy installs from the lock, not a fresh resolve
- [ ] `[SHOULD]` PHP version + required extensions on the server match `composer.json` `require` (pdo, mbstring, openssl, etc.)

## Environment & config
- [ ] `[BLOCKER]` `APP_ENV=production` and `APP_DEBUG=false` — a true `APP_DEBUG` leaks stack traces, env values, and DB creds on any error
- [ ] `[BLOCKER]` `APP_KEY` set, non-default, not committed — required for encryption, signed URLs, encrypted cookies
- [ ] `[BLOCKER]` `.env` not web-accessible and not committed; only `.env.example` in the repo
- [ ] `[BLOCKER]` `APP_URL` set to the real https domain — wrong value breaks signed URLs, queued mail links, asset URLs
- [ ] `[SHOULD]` `TrustProxies` configured for the load balancer / CDN so scheme + client IP are correct
- [ ] `[SHOULD]` `FORCE_HTTPS` / URL scheme forced in production so generated links are https

## Caching & optimization
- [ ] `[BLOCKER]` Config cached in prod (`php artisan config:cache`) — and code never calls `env()` outside config files (env() returns null once config is cached)
- [ ] `[BLOCKER]` Routes cached (`php artisan route:cache`) — verify no closure-based routes (they can't be cached); all routes use controller classes
- [ ] `[SHOULD]` Events and views cached (`event:cache`, `view:cache`); `php artisan optimize` in the deploy step
- [ ] `[SHOULD]` Caches cleared/rebuilt on every deploy (stale config/route cache after a change is a silent prod bug)
- [ ] `[SHOULD]` Autoloader optimized (`--optimize-autoloader` or `composer dump-autoload -o`)

## Routing & HTTP
- [ ] `[BLOCKER]` State-changing web routes go through the `web` middleware group so CSRF is enforced; API routes use token/Sanctum auth, not sessions
- [ ] `[BLOCKER]` No debug/dev routes exposed in prod (Telescope, Horizon, `/_ignition`, ad-hoc `Route::get('debug')`) — or they're gated behind auth + environment
- [ ] `[SHOULD]` Rate limiting applied to auth, API, and expensive routes (`RateLimiter` / `throttle` middleware)
- [ ] `[SHOULD]` Route-model binding + Form Request validation used; no unvalidated `$request->all()` reaching business logic
- [ ] `[SHOULD]` Fallback / 404 route returns a clean page, not a stack trace or default debug screen

## Database & migrations
- [ ] `[BLOCKER]` Migrations run non-interactively on deploy (`php artisan migrate --force`) with a tested rollback path
- [ ] `[BLOCKER]` No destructive migration (drop/rename column) shipped without a data-safe, reversible plan
- [ ] `[SHOULD]` DB indexes exist for columns filtered/sorted at scale; N+1 queries checked (eager-load with `with()`)
- [ ] `[SHOULD]` Seeders/factories are dev-only — not run in production
- [ ] `[NICE]` Read/write connection split configured if using replicas

## Queues, jobs & scheduler
- [ ] `[BLOCKER]` Queue driver is a real backend (redis/database/sqs), not `sync`, in production
- [ ] `[BLOCKER]` Queue workers run under a supervisor (Supervisor / Horizon) that restarts them; workers restarted on deploy (`queue:restart`) so they pick up new code
- [ ] `[SHOULD]` Failed-jobs table + retry/backoff configured; failures monitored, not silently dropped
- [ ] `[SHOULD]` Scheduler entrypoint wired to system cron (`* * * * * php artisan schedule:run`); scheduled tasks verified to fire in prod
- [ ] `[SHOULD]` Long/heavy work is queued, not run in the request cycle; job timeouts + memory limits set
- [ ] `[NICE]` Horizon dashboard protected by auth and gated to admins

## Sessions, cache stores & Redis
- [ ] `[BLOCKER]` Session/cache driver points at a shared store (redis/database), not `array`/`file`, when running multiple app servers — otherwise sessions vanish between requests
- [ ] `[SHOULD]` `SESSION_SECURE_COOKIE=true`, `SESSION_HTTP_ONLY=true`, `SESSION_SAME_SITE` set (lax or stricter)
- [ ] `[SHOULD]` Cache invalidation strategy defined (tags / keys) so deploys and data changes don't serve stale cache
- [ ] `[SHOULD]` Redis secured (auth/password, not exposed publicly) and separate DB indexes for cache vs queue vs session where relevant

## Application security specifics
- [ ] `[BLOCKER]` Mass-assignment guarded — models set `$fillable` (or explicit `$guarded`); no blind `Model::create($request->all())`
- [ ] `[BLOCKER]` No raw SQL built from request input — query builder / Eloquent bindings, never `DB::raw()` string concatenation
- [ ] `[BLOCKER]` Blade output auto-escaped with `{{ }}`; `{!! !!}` only on trusted/sanitized HTML (XSS)
- [ ] `[BLOCKER]` Authorization enforced via Policies / Gates on protected actions — not just route middleware (prevents IDOR)
- [ ] `[SHOULD]` Signed URLs (`URL::signed`) used for public tokenized links (password reset, email verify, unsubscribe) instead of raw IDs
- [ ] `[SHOULD]` Sensitive model attributes use encrypted casts / are in `$hidden`; nothing secret serialized into API responses or logs
- [ ] `[SHOULD]` File uploads validated (mime, size) and stored on a private disk / outside `public/`, not written straight to the web root
- [ ] `[SHOULD]` API tokens (Sanctum/Passport) scoped to least privilege; token abilities checked, not just "is authenticated"

## Logging & observability
- [ ] `[BLOCKER]` Log channel writes somewhere durable and readable in prod (stack/daily/syslog), not the default single file that no one rotates
- [ ] `[SHOULD]` Log level sane for prod (`LOG_LEVEL=warning`/`error`, not `debug`); no request payloads with PII/secrets logged
- [ ] `[SHOULD]` Error tracking wired (e.g. Sentry/Flare) with `APP_DEBUG=false` so users see a clean error page while you get the trace
- [ ] `[NICE]` Health-check route returns app + DB + cache + queue status for the load balancer

## References
- Laravel Deployment guide: https://laravel.com/docs/deployment
- Configuration & environment: https://laravel.com/docs/configuration
- Authentication, authorization & CSRF: https://laravel.com/docs/authentication
- Queues & Horizon: https://laravel.com/docs/queues
- Task scheduling: https://laravel.com/docs/scheduling
- Composer security audit: https://getcomposer.org/doc/03-cli.md#audit
