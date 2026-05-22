# Category: Next.js + Supabase SaaS

Used by: jrs-auto-repair, silver-creek-logistics, manage-worker-bee, future SaaS apps.
This file captures patterns ALL projects in this category share.
Project-specific CLAUDE.md should only document deviations from these defaults.

---

## Platform Decision Gate (Before ANY Deploy Command)

```bash
# Check for Cloudflare bindings (D1/R2/KV)
cat wrangler.jsonc 2>/dev/null | grep -E "d1_databases|r2_buckets|kv_namespaces"
```

No output → marketing/affiliate/SaaS site → deploy to **Vercel**: `vercel --prod`
Has bindings → worker-bee.app subdomain → deploy to **Cloudflare Workers**: `npx wrangler deploy`

Never use `wrangler deploy` on a site without confirmed D1/R2/KV bindings. Wrangler auth cannot be completed non-interactively. See ADR-0014.

If deploy fails → run 5-option check immediately:
```bash
vercel whoami                          # Option 1
env | grep -iE "cloudflare|vercel"     # Option 2
gh repo view --json homepageUrl        # Option 3
# Option 4: mcp__claude_ai_Vercel__deploy_to_vercel
# Option 5: .github/workflows/deploy.yml with VERCEL_TOKEN secret
```

---

## Stack

- Next.js 16 (App Router, RSC by default)
- React 19
- Supabase (Postgres + Auth + Storage)
- TypeScript strict mode
- Tailwind CSS v4
- Deployment: Vercel (default) or Cloudflare Workers (when D1/R2/KV needed)

---

## The Three Supabase Clients — Pick the Right One

| File | Use in | Bypasses RLS? | Reads cookies? |
|---|---|---|---|
| `lib/supabase/client.ts` | Client Components only | No | Yes (browser) |
| `lib/supabase/server.ts` | Server Components, Route Handlers, Server Actions | No | Yes (server) |
| `lib/supabase/admin.ts` | Server-side privileged ops only | YES — service role | No |

**Never import admin.ts from a Client Component.** Service role key leaks to browser bundle. Refactor through a Route Handler.

---

## Dual-Auth Pattern (Admin + Portal)

1. **Admin** (`/admin`) — cookie `admin_session`, signed with `ADMIN_SECRET`, users in `data/admins.json`. Logic in `lib/adminAuth.ts`.
2. **Portal** (`/portal`) — Supabase JWT, refreshed by `proxy.ts` middleware.

**Never mix them.** Admin check: `cookies().get('admin_session')`. Portal check: Supabase `getUser()`. Mixing = silent 401s in prod.

---

## Standard Directory Layout

```
app/
  (site)/          → public marketing pages
  admin/           → admin dashboard (admin auth required)
  portal/          → customer portal (Supabase auth required)
  api/             → Route Handlers
lib/
  supabase/        → the 3 clients above
  shopInfo.ts      → SINGLE source of truth for business info
  articles.ts      → blog content (TS arrays, NOT markdown — ADR-0004)
  howtos.ts        → tutorial content
  adminAuth.ts     → cookie-based admin auth helpers
components/        → React components, shadcn/ui defaults
proxy.ts           → Next.js middleware (Supabase session refresh + redirects)
```

---

## Required Env Vars

```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY    # server-only, service role
ADMIN_SECRET                 # signs admin_session cookie
ANTHROPIC_API_KEY            # if project has AI features
```

Project CLAUDE.md adds anything beyond this list.

---

## Commands

```bash
npm run dev        # localhost:3000 — always use -H 0.0.0.0 for Tailscale
npm run build
npm run lint
npm run test       # Vitest if configured
```

---

## Content Storage Convention

Static content lives in TypeScript arrays, not markdown files. See ADR-0004.

```typescript
// lib/articles.ts
export const articles = [
  { slug, title, excerpt, category, date, readTime, body },
] as const
```

Blog route: always `/blog`, never `/articles` or `/posts`.

---

## SEO

Every project needs:
- `app/sitemap.ts` — generated from articles + howtos + static routes
- `app/robots.ts` — env-gated allow/deny
- JSON-LD: LocalBusiness, FAQPage, Article schemas per route
- Open Graph + Twitter Card meta on every page

Verify after deploy:
```bash
curl -I <url>/sitemap.xml | head -1   # must be 200
curl -I <url>/robots.txt | head -1    # must be 200
```

---

## Failure Modes

- `@supabase/ssr` set to "latest" — pin to a known-good minor version
- `cookies()` inside a Client Component — only `server.ts` knows cookies
- Importing `admin.ts` client-side — service role key leaked, refactor through API
- Forgetting `-H 0.0.0.0` on `next dev` — breaks Tailscale preview pane
- Editing `.env.local` without restarting dev server — Next.js doesn't hot-reload env vars
- `CLOUDFLARE_TOKEN` in env instead of `CLOUDFLARE_API_TOKEN` — wrangler reads the specific name, one-char mismatch silently fails
- Adding new Supabase env var without updating deployment platform — works locally, fails in prod

---

## Testing Conventions

- Vitest, not Jest
- Test files colocated: `lib/foo.ts` + `lib/foo.test.ts`
- Integration tests hit a Supabase test branch, never prod

---

## What Projects in This Category DON'T Do

- No GraphQL — use Supabase REST/RPC
- No Redux/Zustand — RSC + React Context covers it
- No CSS-in-JS — Tailwind v4 only
- No markdown blog files — TS arrays only
- No `pages/` directory — App Router only

---

## Decision Defaults

| User says | Default |
|---|---|
| "add a blog post" | Edit `lib/articles.ts`, no new file |
| "add a route" | `app/(site)/` for public, `app/admin/` or `app/portal/` otherwise |
| "add an API endpoint" | `app/api/<name>/route.ts` |
| "fix auth" | Ask which: `/admin` (cookie) or `/portal` (Supabase JWT) |
| "deploy" | Check platform gate above before any command |
| Mentions animation/scroll | Use `record.js` (video), NOT `screenshot.js` |
