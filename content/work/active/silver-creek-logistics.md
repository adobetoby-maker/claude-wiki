---
ai-first: true
type: project
date: 2026-05-28
tags: [silver-creek, logistics, nextjs, supabase, cloudflare, twilio, dispatch]
status: active
---

# Silver Creek Logistics

## For future Claude
Freight/logistics company site with dispatch automation, customer portal, and QuickBooks integration. Two separate deploy targets: Vercel (main app) + Cloudflare Worker (dispatch cron). Load this when working on SCL, dispatch, invoices, or the CF Worker.

---

## Project
- **Path:** `/Users/drive/silver-creek-logistics`
- **URL:** silvercreeklogistics.worker-bee.app (Vercel)
- **Deploy:** `vercel --prod` (main app) | CF Worker deployed separately from `cloudflare-worker/`
- **Dev:** `npm run dev` (port 3000, `-H 0.0.0.0` required for Tailscale)

## Auth — Dual System, NEVER Mix (ADR-0006)
- `/admin` — cookie `admin_session`, signed with `ADMIN_SECRET`, logic in `lib/adminAuth.ts`
- `/portal` — Supabase JWT via `lib/supabase/server.ts`, refreshed in `proxy.ts`
- Mixing them → silent 401s in prod

## Supabase Clients (same pattern as jrs-auto-repair)
- `lib/supabase/client.ts` — browser only
- `lib/supabase/server.ts` — Server Components / Route Handlers
- `lib/supabase/admin.ts` — privileged server ops ONLY — never import client-side

## Dispatch — Two Separate Systems
- **Vercel cron** (`vercel.json`): daily at `/api/cron/dispatch` — daily summaries
- **Cloudflare Worker** (`cloudflare-worker/silvercreek-dispatch`): every 30 min — real-time dispatch
- `CRON_SECRET` must match in BOTH Vercel env vars AND Cloudflare Worker dashboard — set independently
- When debugging dispatch: determine which system is responsible first

## Static Data Files
- `lib/shopInfo.ts` — business facts (single source of truth)
- `lib/drivers.ts` — driver data for dispatch
- `lib/materials.ts` — freight materials list

## Routes
| Area | Path | Auth |
|---|---|---|
| Marketing | `/(site)/` | Public |
| Calculator | `/(site)/calculator` | Public |
| Admin CRM/invoices/dispatch | `/admin/*` | Cookie |
| Customer portal | `/portal/*` | Supabase JWT |
| Public invoice | `/invoice/[id]` | Public token |

## Integrations
- **Twilio SMS** — dispatch notifications (TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, TWILIO_FROM, TWILIO_DISPATCH_PHONES)
- **QuickBooks OAuth** — invoice sync (QB_CLIENT_ID, QB_CLIENT_SECRET, QB_REDIRECT_URI)
- **Gmail** — email notifications (GMAIL_USER, GMAIL_APP_PASSWORD)

## Failure Patterns
- Mixing /admin and /portal auth → silent 401s
- CRON_SECRET updated in Vercel but not CF Worker → dispatch silently rejected
- `npm run dev` without `-H 0.0.0.0` → Tailscale preview breaks silently
- QB redirect URI mismatch → OAuth fails at callback with no useful error
- Importing admin.ts client-side → service role key exposed

## Decision Defaults
| User says | Default action |
|---|---|
| "fix dispatch" | Ask: Vercel cron or CF Worker? |
| "update driver list" | Edit `lib/drivers.ts` |
| "update business info" | Edit `lib/shopInfo.ts` only |
| "CRON_SECRET not matching" | Update in BOTH Vercel AND CF Worker dashboard |
| "QuickBooks not connecting" | Check QB_CLIENT_ID + redirect URI matches OAuth app |
