---
ai-first: true
type: project
date: 2026-05-28
tags: [jrs, auto-repair, twin-falls, client]
status: active
client: Pablo Zaldavar
---

# Jr.'s Auto Repair

## For future Claude
Client site — Pablo Zaldavar's auto shop in Twin Falls, ID. Full-stack Next.js + Supabase. "Twin Falls" or "Magic Valley" context = this project.

---

## Client
- **Owner:** Pablo Zaldavar
- **Address:** 417 Main Ave E, Twin Falls, ID
- **Phone:** (208) 595-2101
- **Hours:** Mon–Sat 9 AM–5 PM
- **Rating:** 4.8★ / 146 reviews
- **Tagline:** "Honest work, fair prices, done right the first time."

## Project
- **Path:** `/Users/drive/jrs-auto-repair`
- **URL:** jrsautorepair.worker-bee.app (live)
- **Stack:** Next.js 16, Supabase, Tailwind v4, Vitest
- **Deploy:** `npm run dev` (port 3000, `-H 0.0.0.0`)

## Routes
- `/` — landing (Hero, Services, About, Reviews, Contact)
- `/blog` + `/blog/[slug]` — 21 articles (content in `lib/articles.ts`)
- `/how-to` + `/how-to/[slug]` — DIY guides
- `/portal` — customer portal (Supabase JWT auth)
- `/admin` — full admin (invoices, ROs, inventory, users, analytics, marketing, crons)
- `/founders` — founders page

## Critical Rules
- **Blog content:** `lib/articles.ts` TypeScript arrays ONLY — markdown files are silently ignored
- **Auth:** `/admin` = cookie auth. `/portal` = Supabase JWT. NEVER mix — silent session overwrite in prod
- **Supabase clients:**
  - `lib/supabase/server.ts` — Server Components only, reads cookies
  - `lib/supabase/client.ts` — browser only
  - `lib/supabase/admin.ts` — service role, server-side ONLY

## Brand Voice
Mom-and-pop, honest, plain-spoken — never corporate or salesy. "Honest" is the core brand word.
Copy patterns: direct → friendly → specific. Never: "state-of-the-art", "synergy", "solutions provider."
Tagline: "Honest work, fair prices, done right the first time." — the brand in one sentence.

## Failure Patterns
- Adding articles as markdown files → silently ignored, content never shows
- Using `/admin` cookie in `/portal` routes → silent 401s
- Importing admin.ts client-side → service role key in browser bundle

## Supabase
- Project ID: check `.env.local` → `NEXT_PUBLIC_SUPABASE_URL`
- Admin login: adobetoby@gmail.com → password in vault
