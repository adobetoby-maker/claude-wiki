---
ai-first: true
type: project
date: 2026-05-28
tags: [language-learning, tanstack, cloudflare, stripe, supabase, saas]
status: active
---

# Language Lens Elite (app.languagethreshold.com)

## For future Claude
The Language Threshold SaaS app — TanStack Start + Stripe subscriptions. Separate from the three marketing sites (medicalspanish, constructionspanish, juniorlinguist) in [[work/active/language-threshold]]. Load this when working on the authenticated app, Stripe billing, AI tutor routes, or the tab system.

---

## Repo & Deploy
- **Path:** `/Users/drive/language-lens-elite`
- **Live:** https://app.languagethreshold.com (also language-lens-elite.worker-bee.app)
- **Stack:** TanStack Start (React Router v7) + Vite — NOT Next.js, NOT Cloudflare Workers
- **Deploy:** `vercel deploy --prod --yes` (NOT wrangler)
- **Supabase:** `pollhlkgltdkdskdzsgd`

## Commands
```bash
npm run dev          # Vite dev server (auto-picks open port, usually 8080+)
npm run build        # outputs .vercel/output/
npx tsc --noEmit
vercel deploy --prod --yes
```

## Supabase Tables
| Table | Purpose |
|---|---|
| `profiles` | id → auth.users, data JSONB (AppState + Stripe fields) |
| `mission_pins` | missionary location + hometown coords, public read |
| `library_books` | user_id, chapters JSONB |
| `family_groups` | owner_user_id, family_code, mission_pin_id |
| `family_members` | group_id, is_missionary, max 7 (DB trigger) |

## Stripe Billing
- Webhook: https://app.languagethreshold.com/api/stripe-webhook
- Handler: `src/routes/api.stripe-webhook.ts` — 9 events → profiles.data JSONB
- Pricing: Monthly $19/mo · Annual $149/yr · Family $249/yr · 7-day trials
- Payment Links (fallback): Monthly `buy.stripe.com/14A9AVemfg0l1zH8aqbfO08`, Annual `buy.stripe.com/bJe7sN2Dx29v0vD76mbfO09`

## Key Routes
- `src/routes/api.stripe-webhook.ts` — Stripe → Supabase sync
- `src/routes/api.create-checkout.ts` — checkout session (7-day trial)
- `src/routes/pricing.tsx` — 3 tiers + free
- `src/routes/api.tutor.ts`, `api.speak.ts`, `api.discussion.ts` — AI features

## Critical Rules
- Tab system: dual update — `app-state.tsx TabKey` AND `tab-registry.ts TAB_COMPONENTS` — missing either causes TypeScript to miss the tab silently (ADR-0011)
- XP tier ≠ rank tier — completely separate state files
- Client env = `VITE_` prefix via `import.meta.env`; server env = `process.env`
- Stripe webhook 503 = `SUPABASE_SERVICE_ROLE_KEY` or `STRIPE_SECRET_KEY` missing in Vercel

## Known Gaps (as of 2026-05-26)
- `DATABASE_URL` / `DIRECT_URL` still have `[YOUR-PASSWORD]` placeholder — Prisma not yet connected
- `VITE_STRIPE_PRICE_*` not set (pricing falls back to Payment Links — functional)
- Pre-existing TS error in `src/data/curriculum-extended.ts` (non-blocking)

## Env Vars (names only)
```
VITE_SUPABASE_URL, VITE_SUPABASE_PUBLISHABLE_KEY, VITE_SUPABASE_PROJECT_ID
ANTHROPIC_API_KEY, SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY
STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
```
