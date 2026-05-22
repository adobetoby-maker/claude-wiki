# Category: Next.js on Cloudflare Workers

> **Before using this category:** Is this a worker-bee.app project, or does it need D1/R2/KV bindings?
> If not — use `nextjs-vercel-deploy.md` instead. Marketing sites, affiliate sites, and portfolios go on Vercel.
> Wrangler auth cannot be triggered non-interactively. If you hit a wrangler auth wall, see ADR-0014.

Used by: jrs-auto-repair, manage-worker-bee, silver-creek-logistics, language-lens-elite (worker-bee.app subdomains with bindings).
Use ALONGSIDE another category (e.g. nextjs-supabase-saas) — this file covers the deployment specifics only.

## Why Cloudflare Workers Over Vercel

See ADR 0003. Short version: edge-first regions matter for the climb sites (international users), and we want all worker-bee.app subdomains on one DNS infra. Vercel is fine but we standardized on CF for this group.

## The Adapter: @opennextjs/cloudflare

Next.js 16 runs on Cloudflare Workers via `@opennextjs/cloudflare`. This is NOT the legacy `next-on-pages` — different tool.

```bash
npm i -D @opennextjs/cloudflare wrangler
```

```jsonc
// wrangler.jsonc
{
  "name": "<project-name>",
  "main": ".open-next/worker.js",
  "compatibility_date": "2025-10-01",
  "compatibility_flags": ["nodejs_compat"],
  "assets": {
    "directory": ".open-next/assets",
    "binding": "ASSETS"
  }
}
```

## Build + Deploy Commands

```bash
npm run build                              # produces .next/
npx @opennextjs/cloudflare build           # transforms .next/ → .open-next/
npx wrangler deploy                        # ships .open-next/ to CF
```

Combined into a `deploy` script in package.json:

```json
"scripts": {
  "deploy": "npm run build && opennextjs-cloudflare build && wrangler deploy"
}
```

## Environment Variables

Two places to set them:

1. **Local development** — `.env.local` (Next.js reads it normally)
2. **Production** — Cloudflare Worker secrets via `wrangler secret put NAME`

CF Worker secrets are NOT in wrangler.jsonc. Never commit them. Public vars go in `[vars]` block.

```bash
wrangler secret put SUPABASE_SERVICE_ROLE_KEY
wrangler secret put ADMIN_SECRET
wrangler secret put ANTHROPIC_API_KEY
```

## DNS Pattern (worker-bee.app)

All sub-projects deploy under `<name>.worker-bee.app`:

- climb-brasil → climb-brasil.worker-bee.app (also climbbrasil.com via custom domain)
- jrs-auto-repair → jrsautorepair.worker-bee.app
- language-lens-elite → language-lens-elite.worker-bee.app

Custom domains attach via `wrangler custom-domains add`. DNS is managed in the Cloudflare dashboard under the `worker-bee.app` zone.

## What Doesn't Work on Workers

| Next.js feature | Status |
|---|---|
| Image Optimization API | NO — use Cloudflare Images or external CDN |
| Middleware (Edge runtime) | YES — but use Node runtime where possible |
| Route Handlers | YES |
| Server Actions | YES |
| ISR (revalidate) | YES via @opennextjs/cloudflare cache |
| File system writes | NO — use R2 or D1 |
| `fs.readFile` at runtime | NO — bundle the file or use ASSETS binding |

## Storage Bindings

For projects that need it, add bindings in wrangler.jsonc:

```jsonc
"r2_buckets": [{ "binding": "BUCKET", "bucket_name": "..." }],
"d1_databases": [{ "binding": "DB", "database_id": "..." }],
"kv_namespaces": [{ "binding": "KV", "id": "..." }]
```

Access via `getCloudflareContext().env.BUCKET` in server code.

## Cron Triggers

Cloudflare Workers run cron jobs natively. Add to wrangler.jsonc:

```jsonc
"triggers": {
  "crons": ["0 */6 * * *"]
}
```

Then handle in your worker entry. **If using both Vercel and CF crons, CRON_SECRET must match in BOTH dashboards.** This burned us before (see failures.md).

## Local Development

```bash
npm run dev                        # standard Next.js dev — runs in Node, not Workers
npx wrangler dev                   # runs the built worker locally on miniflare
```

Use `npm run dev` for fast iteration. Use `wrangler dev` only when you need to test Worker-specific bindings or cron triggers.

## Logs and Observability

```bash
npx wrangler tail                  # live tail prod logs
npx wrangler tail --format=json    # for piping into jq
```

CF dashboard → Workers → Logs gives the same view with filtering.

## Failure Modes

- Forgetting to set `nodejs_compat` flag → cryptic runtime errors about missing Node APIs
- Setting compatibility_date too old → new Workers features don't work
- Committing `.dev.vars` (CF's local secrets file) → leaks. It's gitignored by default but double-check.
- Mismatched CRON_SECRET between Vercel and CF → cron stops silently with no error
- Using `fs.readFile` at runtime → works in dev (Node), fails in prod (Workers). Inline the file or use ASSETS binding.
- Image optimization with `next/image` default loader → 404s in prod. Use the `loader: 'custom'` pattern or Cloudflare Images.

## Useful Commands Reference

```bash
wrangler whoami                          # check auth
wrangler deploy                          # ship
wrangler tail                            # live logs
wrangler secret list                     # see what's set (not values)
wrangler secret put NAME                 # set a secret
wrangler secret delete NAME              # remove
wrangler kv:key put --binding=KV k v     # KV write
wrangler r2 object put BUCKET/key file   # R2 upload
wrangler d1 execute DB --command "..."   # D1 query
```
