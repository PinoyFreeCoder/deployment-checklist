# Pre-Deployment Checklist — Django (extends base)

Stack-specific pre-deploy checks for **Django** (Python). Run [base.md](./base.md) first; these items are *additional*. Severity labels (`[BLOCKER]` / `[SHOULD]` / `[NICE]`) are defined in the base legend.

Items are stated by intent; the management command / setting in parentheses is the usual way to satisfy them. Start from `python manage.py check --deploy` — it flags most of the config-level BLOCKERs below.

## Dependencies & supply chain
- [ ] `[BLOCKER]` Dependency audit clean (`pip-audit` or `safety check`) — no known-vuln packages, including transitive
- [ ] `[BLOCKER]` Dev-only packages excluded from the prod image (django-debug-toolbar, django-extensions, ipython, pytest) — split requirements or use a `prod` extras group
- [ ] `[SHOULD]` Dependencies pinned with hashes (`pip install --require-hashes` from a compiled `requirements.txt`, or `poetry.lock` / `uv.lock` committed)
- [ ] `[SHOULD]` Python version on the server matches the lockfile / `pyproject.toml`; C-extension wheels (psycopg, Pillow, cryptography) build or install cleanly

## Settings & config
- [ ] `[BLOCKER]` `DEBUG = False` in prod — a true `DEBUG` leaks stack traces, settings, SQL, and the full environment on any error, and disables `ALLOWED_HOSTS` enforcement
- [ ] `[BLOCKER]` `SECRET_KEY` loaded from env, non-default, not committed — rotating it invalidates sessions, password-reset tokens, and signed cookies
- [ ] `[BLOCKER]` `ALLOWED_HOSTS` set to the real domain(s) — empty + `DEBUG=False` returns 400 on every request; `['*']` enables Host-header poisoning
- [ ] `[BLOCKER]` `CSRF_TRUSTED_ORIGINS` includes the real https origin(s) (Django 4+ requires scheme) — otherwise POST/PUT forms and the admin fail with 403
- [ ] `[SHOULD]` Settings split per environment (base/prod/dev) or driven entirely by env vars; no secrets hardcoded in `settings.py`
- [ ] `[SHOULD]` `SECURE_PROXY_SSL_HEADER` set when behind a TLS-terminating proxy/CDN so Django sees requests as https (else infinite redirect loop or wrong scheme in URLs)

## HTTPS, cookies & security middleware
- [ ] `[BLOCKER]` `SecurityMiddleware` enabled with `SECURE_SSL_REDIRECT = True`, `SESSION_COOKIE_SECURE = True`, `CSRF_COOKIE_SECURE = True` — cookies must not travel over http
- [ ] `[BLOCKER]` `django.middleware.csrf.CsrfViewMiddleware` present and not bypassed; no blanket `@csrf_exempt` on state-changing views
- [ ] `[SHOULD]` HSTS set (`SECURE_HSTS_SECONDS` ≥ 31536000, `SECURE_HSTS_INCLUDE_SUBDOMAINS`, `SECURE_HSTS_PRELOAD`) — only after https is confirmed working, it's hard to undo
- [ ] `[SHOULD]` `SECURE_CONTENT_TYPE_NOSNIFF = True`, `X_FRAME_OPTIONS = 'DENY'` (or CSP frame-ancestors), `SESSION_COOKIE_HTTPONLY = True`, `SESSION_COOKIE_SAMESITE` set

## Static & media files
- [ ] `[BLOCKER]` Static files collected and served by the web server / CDN (`python manage.py collectstatic --noinput`) — Django does not serve static in prod; WhiteNoise or an upstream server does
- [ ] `[BLOCKER]` User-uploaded `MEDIA` served from outside the app, never executed as code, and not mixed into `STATIC_ROOT`
- [ ] `[SHOULD]` `MEDIA_ROOT` / uploads validated for type + size; untrusted files stored on a separate domain or object storage, not the app host
- [ ] `[NICE]` Hashed/manifest static storage enabled for cache-busting (`ManifestStaticFilesStorage` or WhiteNoise compressed manifest)

## Database & migrations
- [ ] `[BLOCKER]` Migrations applied non-interactively on deploy (`python manage.py migrate --noinput`) with a tested rollback / forward-fix path
- [ ] `[BLOCKER]` No destructive migration (drop/rename column, data migration) shipped without a reversible, zero-downtime plan (expand/contract for rolling deploys)
- [ ] `[BLOCKER]` `python manage.py makemigrations --check --dry-run` is clean — no un-committed model changes that would diverge from the DB
- [ ] `[SHOULD]` `CONN_MAX_AGE` (or an external pooler like PgBouncer) set so each request doesn't open a fresh connection; `ATOMIC_REQUESTS` chosen deliberately
- [ ] `[SHOULD]` Indexes exist for fields filtered/ordered at scale; N+1 checked (`select_related` / `prefetch_related`); `.only()`/`.defer()` for heavy rows
- [ ] `[NICE]` Read replica wired via a database router if used

## Views, ORM & app security
- [ ] `[BLOCKER]` No raw SQL built from request input — ORM / parameterized `.raw(sql, params)`, never f-string/`%`-formatted queries or `.extra()` with user data
- [ ] `[BLOCKER]` Object-level authorization enforced per view (`get_object_or_404` scoped to `request.user`, or permission checks) — a logged-in user must not fetch another user's row by ID (IDOR)
- [ ] `[BLOCKER]` Templates rely on Django auto-escaping; `|safe`, `mark_safe`, and `format_html` only on trusted/sanitized content (XSS)
- [ ] `[SHOULD]` Forms / DRF serializers validate input; no building model instances straight from unvalidated POST data; file-upload fields validate content type + size
- [ ] `[SHOULD]` DRF defaults locked down — `DEFAULT_PERMISSION_CLASSES` is not `AllowAny`, `DEFAULT_THROTTLE_CLASSES`/rates set, browsable API disabled or auth-gated in prod
- [ ] `[SHOULD]` Rate limiting / throttling on auth and expensive endpoints (DRF throttling, django-ratelimit, or upstream)

## Django admin
- [ ] `[BLOCKER]` Admin not at the default `/admin/` path, or protected by an extra layer (IP allowlist / SSO / VPN); strong unique superuser credentials
- [ ] `[SHOULD]` Admin reachable only over https with secure session cookies; staff accounts use 2FA where possible
- [ ] `[NICE]` Admin actions that bulk-delete or export PII are logged / restricted

## Server, WSGI/ASGI & tasks
- [ ] `[BLOCKER]` App runs under a production server (gunicorn/uvicorn) behind nginx/a proxy — never `manage.py runserver` in prod
- [ ] `[BLOCKER]` Celery/background workers (and beat scheduler) run under a supervisor that restarts them, and are restarted on deploy to pick up new code; broker (Redis/RabbitMQ) is authenticated and not publicly exposed
- [ ] `[SHOULD]` Worker/web process counts and timeouts tuned for the host; long/heavy work is queued, not run in the request cycle
- [ ] `[SHOULD]` Cache backend is a shared store (Redis/Memcached), not `LocMemCache`, when running multiple processes/hosts; sessions use that backend or the DB
- [ ] `[SHOULD]` `DATA_UPLOAD_MAX_MEMORY_SIZE` / `DATA_UPLOAD_MAX_NUMBER_FIELDS` set to sane limits to blunt large-payload DoS

## Logging & observability
- [ ] `[BLOCKER]` `LOGGING` configured to write durable, readable logs in prod (console → aggregator, or syslog) — not swallowed; `ADMINS` + mail or an error tracker get unhandled 500s
- [ ] `[SHOULD]` Error tracking wired (e.g. Sentry) with `DEBUG=False`, so users see a clean 500 page while you get the trace; PII/secrets scrubbed from events
- [ ] `[SHOULD]` No request bodies, passwords, tokens, or session keys logged; log level is `WARNING`/`ERROR` in prod, not `DEBUG`
- [ ] `[NICE]` Health-check endpoint reports app + DB + cache + broker status for the load balancer

## References
- Deployment checklist (official): https://docs.djangoproject.com/en/stable/howto/deployment/checklist/
- Security in Django: https://docs.djangoproject.com/en/stable/topics/security/
- `check --deploy` system checks: https://docs.djangoproject.com/en/stable/ref/checks/#security
- Static files deployment: https://docs.djangoproject.com/en/stable/howto/static-files/deployment/
- Settings reference: https://docs.djangoproject.com/en/stable/ref/settings/
- pip-audit: https://pypi.org/project/pip-audit/
