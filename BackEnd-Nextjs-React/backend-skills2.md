# Next.js 16 + Prisma 7 + NextAuth + TypeScript + PostgreSQL + Tailwind

## Secure Web App Design Guidelines (Best Practices)

> Stack assumptions: Next.js 16 (App Router), Prisma 7, NextAuth (Auth.js), PostgreSQL, Tailwind, TypeScript.

---

## 1) Core Security Principles (Non-Negotiables)

### Default to **server-first**

- Keep secrets, tokens, DB access, and privileged logic **only on the server** (Server Components, Route Handlers, Server Actions).
- Never put secrets in `NEXT_PUBLIC_*`.
- Treat client code as attacker-controlled.

### Least privilege everywhere

- DB user has minimal permissions (no superuser; restricted schema access).
- Separate roles if needed: `app_user` (runtime) vs `migration_user` (migrations only).
- Restrict API keys per environment (dev/stage/prod).

### Strong boundaries

- Validate all inputs at the edge of the system (Route Handlers / Server Actions).
- Enforce authorization **server-side**, not via UI checks.

---

## 2) Recommended Folder Structure (App Router)

A practical structure that scales and keeps security boundaries clear:

.
├─ app/
│ ├─ (public)/
│ │ ├─ page.tsx
│ │ └─ layout.tsx
│ ├─ (auth)/
│ │ ├─ login/page.tsx
│ │ └─ layout.tsx
│ ├─ (app)/
│ │ ├─ dashboard/page.tsx
│ │ ├─ settings/page.tsx
│ │ └─ layout.tsx
│ ├─ api/
│ │ ├─ auth/[...nextauth]/route.ts
│ │ ├─ health/route.ts
│ │ └─ webhooks/
│ │ └─ provider-x/route.ts
│ ├─ actions/
│ │ ├─ user.actions.ts
│ │ └─ billing.actions.ts
│ ├─ error.tsx
│ ├─ not-found.tsx
│ ├─ layout.tsx
│ └─ middleware.ts
├─ components/
│ ├─ ui/ # dumb/presentational components
│ ├─ forms/
│ └─ shared/
├─ lib/
│ ├─ auth/
│ │ ├─ config.ts # NextAuth config
│ │ ├─ session.ts # server helpers
│ │ └─ permissions.ts # RBAC/ABAC checks
│ ├─ db/
│ │ ├─ prisma.ts # Prisma singleton
│ │ └─ queries/ # read models (safe selects)
│ ├─ security/
│ │ ├─ csrf.ts
│ │ ├─ rate-limit.ts
│ │ ├─ headers.ts
│ │ └─ validation.ts
│ ├─ cache/
│ │ ├─ tags.ts
│ │ └─ revalidate.ts
│ └─ utils/
├─ prisma/
│ ├─ schema.prisma
│ ├─ migrations/
│ └─ seed.ts
├─ styles/
│ └─ globals.css
├─ types/
│ └─ index.d.ts
├─ scripts/
│ └─ ci/
├─ .env.example
├─ next.config.js
└─ package.json

**Rules of thumb**

- `app/api/**` = external interface. Always validate + auth + rate-limit.
- `app/actions/**` = server-only business actions. Validate + authorize.
- `lib/**` = reusable server utilities. Keep security helpers here.

---

## 3) Authentication & Session Security (NextAuth)

### Session strategy

- Prefer **JWT sessions** only if you fully understand token invalidation tradeoffs.
- For higher security and easy revocation, prefer **database sessions** (session stored server-side).

### Cookie hardening

Ensure cookies are:

- `HttpOnly` (prevents JS access)
- `Secure` (HTTPS only in prod)
- `SameSite=Lax` or `Strict` (balance UX and CSRF resistance)
- Use a strong `NEXTAUTH_SECRET`.

### Protect routes properly

- Use middleware sparingly (cheap checks + redirects).
- Do **real authorization** in server code (Route Handlers / Server Actions).

### RBAC/ABAC

Centralize permission logic in `lib/auth/permissions.ts`:

- `canReadProject(user, project)`
- `canEditBilling(user, org)`
  Never spread permission logic across pages/components.

### Account linking safety

If using OAuth:

- Verify email domain / verification if required by your app.
- Guard against account takeover via provider mismatch (link accounts only intentionally).

---

## 4) Database Security (PostgreSQL + Prisma)

### Prisma client setup (avoid connection storms)

- Use a singleton Prisma client in `lib/db/prisma.ts` (especially for dev + hot reload).
- In serverless environments, use a pooling solution (PgBouncer / provider pooling).

### Use explicit selects (avoid leaking sensitive columns)

Create “safe query” helpers in `lib/db/queries/`:

- Never `include: { ... }` blindly.
- Prefer `select` with only fields you need.

### Multi-tenant safety (if applicable)

- Enforce tenant scoping at the query layer:
  - Always filter by `orgId` / `tenantId`.
- Consider PostgreSQL Row Level Security (RLS) for hard guarantees (advanced but strong).

### Prevent injection

Prisma protects against classic SQL injection when using Prisma query APIs.
**Still avoid**:

- `$queryRawUnsafe`
- string concatenation in raw SQL
  If you must use raw SQL, use parameterized `$queryRaw`.

### Migrations

- Run migrations in CI/CD using a dedicated migration role.
- Don’t ship apps that auto-migrate on startup.

---

## 5) Input Validation & Output Encoding (XSS prevention)

### Validate every external input

- Route Handlers: validate `params`, `searchParams`, `headers`, `body`.
- Server Actions: validate action arguments.
  Use a single validation strategy (commonly Zod) in `lib/security/validation.ts`.

### Encode output by default

- React escapes strings by default: keep it that way.
- Avoid `dangerouslySetInnerHTML`.
  If you must render rich text:
- sanitize server-side with a proven sanitizer
- store sanitized version (or sanitize at render time)

---

## 6) CSRF, CORS, and Webhooks

### CSRF

- If using cookie-based auth for state-changing requests, you need CSRF protection.
- NextAuth provides CSRF protection for its own flows; for your custom endpoints:
  - Use SameSite cookies + CSRF tokens for unsafe methods (POST/PUT/PATCH/DELETE).

### CORS

- Prefer **same-origin** APIs.
- If you must allow cross-origin:
  - do not use `Access-Control-Allow-Origin: *` with credentials
  - allowlist exact origins
  - restrict methods + headers

### Webhooks (high risk)

- Verify signatures (HMAC) using a raw request body.
- Use idempotency keys / event IDs to prevent replay.
- Rate-limit and log.

---

## 7) Rate Limiting, Abuse Controls, and DoS

Apply rate limits to:

- login / password reset
- signup / invite endpoints
- any expensive queries
- public search endpoints

Implementation options:

- Redis-based limiter (best)
- Provider edge rate limit (if available)
- Simple in-memory limiter only for dev (not reliable in prod)

Also:

- Put max sizes on payloads
- Pagination everywhere
- Timeouts for external calls

---

## 8) HTTP Security Headers (must-have baseline)

Set headers globally (via `next.config.js` or middleware where appropriate):

- `Strict-Transport-Security` (HSTS) in prod
- `Content-Security-Policy` (CSP) — start strict, relax carefully
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy` (restrict camera/mic/geolocation if unused)
- `X-Frame-Options` (or CSP `frame-ancestors`) to prevent clickjacking

**CSP tip**

- Avoid `unsafe-inline` if possible.
- If you use 3rd-party scripts (analytics), pin sources tightly.

---

## 9) Safe Data Fetching Patterns (Next.js 16)

### Prefer Server Components for reads

- Load data directly from Prisma in server components where possible.
- Keep read logic in `lib/db/queries`.

### Use Route Handlers for external API contracts

- When clients need fetchable endpoints (mobile apps, webhooks, public API), use `app/api/**`.
- Always:
  - authenticate
  - authorize
  - validate
  - rate-limit
  - return minimal data

### Server Actions for mutations

- Mutations should live in `app/actions/**`.
- Always validate + authorize inside the action.
- Avoid passing raw DB models back to the client; map to DTOs.

### Avoid client-side fetching for privileged data

Client-side fetching increases:

- attack surface (token exposure, CORS/CSRF mistakes)
- data leakage risks
  If you do client fetching:
- use short-lived access patterns
- ensure endpoint returns only what’s needed

---

## 10) Caching & Revalidation (Security-aware)

### Know what is safe to cache

- Public, non-user-specific content: cache freely.
- User-specific or sensitive content: **do not** cache shared.
  - Use `cache: 'no-store'` for personalized data fetches.
  - For pages showing private data, ensure they are dynamic and not statically cached.

### Recommended approach

- Public pages:
  - Use ISR / revalidate with tags
- Authenticated dashboards:
  - Prefer dynamic rendering + `no-store`
- Mutation flows:
  - After a write, call tag-based revalidation for public caches
  - Never revalidate into a shared cache with private data

### Tagging

Centralize cache tags in `lib/cache/tags.ts`:

- `tagUser(id)`, `tagProject(id)`, etc.

---

## 11) SEO Optimization (without hurting security)

### Metadata (App Router)

- Use `export const metadata` and dynamic metadata for content pages.
- Ensure correct:
  - `title`, `description`
  - Open Graph + Twitter cards
  - canonical URLs
  - robots directives

### Indexing rules

- Public content: indexable.
- Auth pages and private dashboards:
  - `robots: { index: false, follow: false }`
  - Avoid leaking internal URLs in sitemaps.

### Sitemaps & robots.txt

- Generate `sitemap.xml` for public pages.
- Keep private routes out.

### Performance

- Use `next/image`
- Lazy-load non-critical components
- Avoid heavy client JS on SEO pages (keep them mostly Server Components)

---

## 12) Secrets & Environment Management

### Must-haves

- `.env.example` checked in; `.env` never checked in.
- Separate env vars for dev/stage/prod.
- Rotate secrets if exposed.

### Typical secrets

- `DATABASE_URL`
- `NEXTAUTH_SECRET`
- OAuth client secrets
- Webhook signing secrets
- Encryption keys (if encrypting PII)

---

## 13) Logging, Monitoring, and Error Handling (secure)

### Avoid sensitive logs

- Never log tokens, passwords, auth headers, full request bodies, or PII.
- Mask emails/IDs if logs are shared.

### Errors

- Return generic errors to clients.
- Log detailed errors server-side (with correlation IDs).

### Audit trails

For sensitive apps:

- Record who did what, when:
  - login events
  - permission changes
  - billing changes
  - data export actions

---

## 14) Dependency & Supply Chain Security

- Enable `npm audit` in CI (but review false positives).
- Pin versions where appropriate.
- Use Dependabot/Renovate + review diffs.
- Avoid random packages for auth/crypto.
- Run `lint`, `typecheck`, and tests in CI.

---

## 15) Common “Gotchas” (What usually gets teams owned)

- Trusting client-side role checks (must enforce on server)
- Caching private data in shared caches
- Over-broad CORS with credentials
- Missing webhook signature verification
- Returning full Prisma models (leaks hidden fields)
- Using `$queryRawUnsafe`
- Storing secrets in `NEXT_PUBLIC_*`
- Forgetting HSTS / CSP / frame-ancestors
- Weak rate limits on auth endpoints

---

## 16) Practical Secure Defaults Checklist

### App

- [ ] All mutations go through Server Actions or authenticated API routes
- [ ] Central validation (Zod) used everywhere
- [ ] Central authorization helpers (RBAC/ABAC)
- [ ] Rate limiting on auth + expensive routes
- [ ] Webhook signature verification + replay protection

### Headers

- [ ] HSTS
- [ ] CSP (tight)
- [ ] frame-ancestors / X-Frame-Options
- [ ] nosniff, referrer-policy, permissions-policy

### DB

- [ ] Least-privilege DB role
- [ ] Explicit `select` patterns
- [ ] Tenant scoping enforced
- [ ] No unsafe raw SQL

### Auth

- [ ] Secure cookies, strong secret
- [ ] Session invalidation strategy decided
- [ ] Email verification / account linking rules defined

### SEO

- [ ] Private routes noindex
- [ ] Public pages metadata + canonical
- [ ] Sitemaps exclude private URLs

---

## Suggested “Golden Rules” for Your Code Reviews

1. “Where is validation happening?”
2. “Where is authorization enforced server-side?”
3. “Could this response leak data if cached/shared?”
4. “Can this endpoint be abused at scale?”
5. “Do we log anything sensitive?”

---
