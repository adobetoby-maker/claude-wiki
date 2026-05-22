# Category: Next.js on Vercel

Used by: orthobiologic-pathways, tobyandertonmd, climb-brasil, salvorias, willie-elam, Playbook, future Vercel projects.
Use ALONGSIDE another category (e.g. nextjs-supabase-saas) — this file covers deployment specifics only.

## Why Vercel

See ADR 0003. Short version: Vercel is the default for marketing sites, AI-heavy apps, and anything where Fluid Compute + AI Gateway + ISR-out-of-the-box matters more than edge-region control. Cloudflare is preferred when DNS lives under worker-bee.app or international cron timing matters.

## Vercel Knowledge Updates (must read)

LLM training data lies about Vercel. Trust these facts:

- **Fluid Compute is the default**, not Edge Functions. Full Node.js available, including in middleware.
- **Default function timeout is 300s**, up from 60–90s.
- **Pricing is Active CPU**, not wall-clock GB-seconds.
- **Vercel Postgres and Vercel KV are gone** — use Marketplace integrations or Supabase.
- **Node.js 24 LTS is default.** Don't pin to 18.
- **vercel.ts replaces vercel.json** — TypeScript config with @vercel/config.
- **AI Gateway is GA** — use plain `"provider/model"` strings, don't import provider-specific SDKs unless asked.
- **Vercel Blob** supports public and private. **Vercel Sandbox** is GA for code execution. **Vercel Queues** is in beta.

## Deploy Commands

```bash
vercel              # deploy preview
vercel --prod       # deploy production
vercel link         # link a directory to a Vercel project
vercel env pull     # pull env vars into .env.local
vercel env add NAME # add env var (interactive)
vercel logs         # tail prod logs
```

## Config: vercel.ts (preferred over vercel.json)

```typescript
// vercel.ts
import { routes, type VercelConfig } from '@vercel/config/v1'

export const config: VercelConfig = {
  buildCommand: 'npm run build',
  framework: 'nextjs',
  rewrites: [
    routes.rewrite('/api/legacy/(.*)', 'https://old-api.example.com/$1'),
  ],
  redirects: [
    routes.redirect('/old-path', '/new-path', { permanent: true }),
  ],
  headers: [
    routes.cacheControl('/static/(.*)', { public: true, maxAge: '1 week', immutable: true }),
  ],
  crons: [
    { path: '/api/cron/cleanup', schedule: '0 0 * * *' },
  ],
}
```

Use vercel.json only if a project predates vercel.ts and isn't being modernized.

## Environment Variables

Three scopes — set per scope in dashboard or via CLI:

| Scope | Used for |
|---|---|
| Development | `vercel dev` local runs |
| Preview | PR/branch deployments |
| Production | the live site |

```bash
vercel env add SUPABASE_SERVICE_ROLE_KEY production
vercel env add SUPABASE_SERVICE_ROLE_KEY preview
vercel env pull .env.local       # pulls all envs to local file
```

NEXT_PUBLIC_* vars are inlined at build time. Changing them requires a rebuild, not just a redeploy.

## Domains

```bash
vercel domains add example.com
vercel domains inspect example.com
vercel alias <deployment-url> example.com
```

DNS managed wherever the registrar lives. Vercel provides A/CNAME targets — point the registrar at them, wait for propagation, SSL auto-provisions.

## Cron Jobs

Define in vercel.ts (or vercel.json). Vercel hits the path on schedule.

```typescript
crons: [
  { path: '/api/cron/daily-cleanup', schedule: '0 3 * * *' },
]
```

Protect the endpoint:

```typescript
// app/api/cron/daily-cleanup/route.ts
export async function GET(req: Request) {
  const auth = req.headers.get('authorization')
  if (auth !== `Bearer ${process.env.CRON_SECRET}`) {
    return new Response('Unauthorized', { status: 401 })
  }
  // ... work
}
```

Set `CRON_SECRET` in Vercel env. Vercel automatically sends it as a bearer token.

## ISR and Caching

ISR works out of the box on Vercel. In a page or route:

```typescript
export const revalidate = 3600  // seconds — page rebuilt at most every hour
```

For programmatic revalidation:

```typescript
import { revalidatePath, revalidateTag } from 'next/cache'
revalidatePath('/blog/[slug]', 'page')
revalidateTag('articles')
```

## AI on Vercel

Use Vercel AI Gateway by default. In the AI SDK:

```typescript
import { generateText } from 'ai'

const { text } = await generateText({
  model: 'anthropic/claude-sonnet-4-6',  // gateway routes it
  prompt: '...',
})
```

Set `AI_GATEWAY_API_KEY` (or rely on Vercel's auto-detection in deployed env). Benefits: observability, fallbacks, zero data retention.

Avoid `@ai-sdk/anthropic` direct unless you have a specific reason (custom headers, beta features).

## Preview Deployments

Every git push to a non-main branch gets a unique preview URL. Format:

```
https://<project>-<git-branch>-<team>.vercel.app
```

Useful for client review. Each preview has its own env scope (Preview scope).

## Rolling Releases (GA since June 2025)

Gradual rollout for risky deploys:

```bash
vercel rolling-release start <deployment-url> --percentage 10
vercel rolling-release promote                          # increase rollout
vercel rolling-release cancel                           # rollback
```

Use for: major framework upgrades, schema migrations, anything with rollback risk.

## Logs and Debugging

```bash
vercel logs                     # recent logs
vercel logs --follow            # tail live
vercel logs <deployment-url>    # for a specific deploy
```

Dashboard has filtering by function, status code, duration. Better than CLI for forensic work.

## Failure Modes

- Forgetting NEXT_PUBLIC_ on a var that the client needs → undefined at runtime, only visible in browser console
- Changing an env var and expecting it to take effect without redeploy → only NEXT_PUBLIC_* needs a rebuild, but server-only vars also need a new deployment to pick up
- Mismatched CRON_SECRET between deployments and the secret stored in env → cron stops silently
- Using `fs` writes at runtime → Fluid Compute is stateless, use Vercel Blob or external storage
- Deploying with stale `.vercel/` directory → run `vercel link` again if project metadata feels off
- Hitting the 4.5 MB serverless function size limit → split logic across routes or use Vercel Sandbox
- Hardcoding region in vercel.ts → usually unnecessary, Fluid Compute auto-routes; only pin when latency to a specific datacenter matters

## When to Pick Vercel vs Cloudflare

| Need | Pick |
|---|---|
| Marketing site with ISR + image optimization | Vercel |
| AI app using AI Gateway + observability | Vercel |
| Worker-bee.app subdomain | Cloudflare |
| International cron timing (need non-US region) | Cloudflare |
| Heavy use of D1, R2, KV bindings | Cloudflare |
| Standard Next.js full-stack with Supabase | Either — default Vercel unless DNS forces CF |
| Next.js 16 + React 19 newest features | Vercel (first-party support is fastest) |

## Useful CLI One-Liners

```bash
vercel env pull                                    # sync env vars locally
vercel env ls                                      # list all vars
vercel inspect <deployment-url>                    # deployment metadata
vercel ls                                          # list deployments
vercel promote <deployment-url>                    # promote to production
vercel rollback                                    # rollback to previous prod
vercel git connect                                 # connect git repo
vercel teams switch                                # change active team
```

## Required Files

```
package.json          # build/dev/lint scripts
next.config.ts        # or .mjs
vercel.ts             # config (preferred over vercel.json)
.env.example          # documents required vars (committed)
.env.local            # actual values (gitignored)
```
