---
ai-first: true
type: project
date: 2026-05-28
tags: [saas, localization, nextjs, supabase, stripe, stalled]
status: archived
---

# Pronto — Localization SaaS

## For future Claude
Localization SaaS competing with forthwith.dev. CLI + web UI for i18n. Targets WordPress/Webflow/Shopify builders + Spanish/Japanese markets. Phase 1 deployed but stalled on Supabase migration (IPv6 block) and Stripe integration. Load this if the Pronto project resumes.

---

## What It Is
forthwith.dev competitor: CLI scan→translate→ship + web UI dashboard for non-devs. Adds site builders (WordPress, Webflow, Shopify, Squarespace, Framer, Wix) that forthwith skips.

## Repos
- `pronto-en` → English. Local: `/Users/drive/pronto-en`. Site: `pronto-en.worker-bee.app`
- `pronto-es` → Spanish native codebase (LATAM/Spain market)
- `pronto-jp` → Japanese native codebase (Japan → US market)
- Supabase ref: `jqzgsbdwwymihyyfzlvs`

## Current State (2026-05-10)
- Live Vercel deploy: 200 OK (deployment protection disabled)
- Supabase schema migration: WRITTEN but not applied (IPv6 CLI block)
  - Apply at: https://supabase.com/dashboard/project/jqzgsbdwwymihyyfzlvs/sql/new
  - File: `/Users/drive/pronto-en/supabase/migrations/20260510000000_pronto_schema.sql`
- Stripe: not wired (needs STRIPE_SECRET_KEY, publishable key, webhook secret, 3 Price IDs)

## Pricing Tiers
- Flex: $0 setup, $0.79/1K words (pay-as-you-go)
- Studio: $49/mo, 150K words, $0.55/1K overage
- Agency: $149/mo, 500K words, $0.40/1K overage + multi-project + priority

## Design System
- Background: `#09090B` | Cards: `#18181B` | Border: `#27272A`
- Primary: `#6366F1` (indigo-500) | Text: `#FAFAFA`
- Fonts: Inter + JetBrains Mono — dark terminal, Linear/Vercel aesthetic

## Stripe Reference Pattern
From `adobetoby-maker/growyournumber` (Supabase Edge Functions + Deno):
- `create-checkout` → find/create Stripe customer → checkout.session → return URL
- `check-subscription` → list subscriptions → check active → return tier
- `customer-portal` → billingPortal.session → return URL
- API version: `2025-08-27.basil`
