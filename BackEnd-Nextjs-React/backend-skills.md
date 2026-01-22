# Next.js 16 + Prisma 7 + NextAuth + TypeScript + PostgreSQL + Tailwind

## Secure Web App Design Guidelines (Best Practices)

> Goal: a modern, fast app that is hard to break (auth, data access, input handling, secrets, SSR/CSR boundaries, and supply chain).

---

## 1) Baseline Security Principles (non-negotiables)

### Keep secrets on the server. Always.

- Never expose DB credentials, API keys, OAuth secrets, or internal endpoints to the client.
- Only read secrets from server-side code (`app/`, `server/`, `route.ts`, Server Components, Server Actions).
- Use separate env files per environment and never commit them:
  - `.env` (local), `.env.production` (CI), and managed secrets in your host (DO / Vercel / etc).

### “Trust nothing” rules

- Validate **every** external input: query params, body, cookies, headers, webhooks.
- Encode output to prevent XSS; avoid dangerous HTML injection.
- Enforce authorization in the **data layer**, not just the UI.

### Least privilege everywhere

- DB user for the app should not be a superuser; only grant needed privileges.
- Separate roles/users for migrations vs runtime if possible.

---

## 2) Recommended Folder Structure (App Router)

This structure separates **UI** from **server-only** and makes it hard to accidentally import secrets into client code.

.
├─ app/
│ ├─ (public)/
│ │ ├─ page.tsx
│ │ └─ layout.tsx
│ ├─ (app)/
│ │ ├─ dashboard/
│ │ │ ├─ page.tsx
│ │ │ ├─ loading.tsx
│ │ │ └─ error.tsx
│ │ └─ layout.tsx
│ ├─ api/
│ │ ├─ auth/[...nextauth]/route.ts
│ │ ├─ health/route.ts
│ │ └─ webhooks/stripe/route.ts
│ ├─ sitemap.ts
│ ├─ robots.ts
│ ├─ globals.css
│ └─ middleware.ts
│
├─ server/ # server-only (NO client imports)
│ ├─ auth/
│ │ ├─ options.ts # NextAuth config
│ │ ├─ permissions.ts # RBAC helpers
│ │ └─ session.ts # getServerSession wrappers
│ ├─ db/
│ │ ├─ prisma.ts # singleton Prisma client
│ │ └─ queries/ # data access functions
│ ├─ services/ # business logic (payments, email, etc.)
│ ├─ security/
│ │ ├─ rate-limit.ts
│ │ ├─ headers.ts
│ │ └─ audit.ts
│ └─ validators/ # zod schemas
│
├─ components/
│ ├─ ui/ # shadcn style components
│ └─ shared/
│
├─ lib/
│ ├─ env.ts # typed env parsing (zod)
│ ├─ utils.ts
│ └─ constants.ts
│
├─ prisma/
│ ├─ schema.prisma
│ ├─ migrations/
│ └─ seed.ts
│
├─ public/
├─ types/
├─ tests/
│ ├─ unit/
│ └─ e2e/
│
├─ tailwind.config.ts
├─ next.config.ts
└─ package.json

**Rules to enforce this:**

- Put `"server-only"` at the top of files that must never run in the browser:
  - `server/db/prisma.ts`, `server/services/*`, etc.
- Use ESLint rules (or convention) to forbid importing `server/*` from client components.

---

## 3) Authentication & Session Security (NextAuth/Auth.js)

### Session strategy

- Prefer **database-backed sessions** (adapter) for revocation + better control.
- If you use JWT sessions: set short expiration + rotate and consider refresh strategy.

### Cookie hardening (critical)

In your NextAuth options:

- `sessionToken` / cookies should be:
  - `httpOnly: true`
  - `secure: true` in production
  - `sameSite: "lax"` (or `"strict"` if your flow allows)
- Set a strong `NEXTAUTH_SECRET`.

### CSRF

- NextAuth already includes CSRF protection for its flows.
- For your **own** mutation endpoints (Route Handlers / webhooks):
  - If called from browser: require same-site cookies + CSRF token OR use Server Actions (preferred).
  - If called server-to-server: use HMAC signatures (Stripe style) or long random tokens.

### Authorization (don’t confuse with authentication)

- Never trust `role` coming from client.
- Always enforce authorization **server-side**, close to queries.
- Implement a permissions layer:
  - `server/auth/permissions.ts` → `can(user, "read:project", projectId)`.

---

## 4) Data Access & Prisma 7 Security

### Prisma client singleton (prevents connection storms)

`server/db/prisma.ts` (Node runtime):

```ts
import "server-only";
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === "development" ? ["warn", "error"] : ["error"],
  });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```
