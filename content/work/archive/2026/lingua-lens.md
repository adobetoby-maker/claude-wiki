---
ai-first: true
type: project
date: 2026-05-28
tags: [language-learning, nextjs, supabase, archived, superseded]
status: archived
---

# LinguaLens (Archived)

## For future Claude
The original Next.js 16 rewrite of language-lens-elite.lovable.app. Superseded by [[work/active/language-threshold]] (TanStack Start / language-lens-elite). Load this only if referencing specific component patterns from this codebase — matchmaking, flashcard, WritingStudio implementations.

---

## What It Was
Full rewrite of `language-lens-elite.lovable.app` (was Lovable/TanStack/Gemini) → Next.js 16.2.4 App Router + Supabase + Anthropic SDK.

- **Path:** `/Users/drive/lingua-lens`
- **Supabase:** `cadyxjryolayvxcynkeb`
- **Stack:** Next.js 16.2.4, shadcn base-ui variant, useReducer + localStorage

## Superseded By
[[work/active/language-threshold]] — TanStack Start app at app.languagethreshold.com / language-lens-elite.worker-bee.app

## Useful Patterns From This Codebase

### Matchmaking (Supabase polling)
- `match_queue` table: INSERT → poll every 1s up to 5s → match real user OR fallback to NPC
- `cancelledRef` guards against stale async resolves after modal close
- `supabase = useMemo(() => createClient(), [])` — stable client

### FlashcardOverlay
- Shuffled single-pass deck, tap to flip
- Got it (+5 XP) / Missed tracking
- Completion screen with accuracy % + total XP

### WritingStudio
- Writing type selector (module-contextual)
- Clickable grammar corrections: highlights phrase via `setSelectionRange`
- Apply All Fixes button
- History drawer (localStorage, max 20 entries)

### LookupContext (two-layer persistence)
- localStorage (instant) + Supabase `saved_words` (durable)
- Auth listener merges Supabase saved words on sign-in

## Pending Items (never completed — use as reference for language-lens-elite)
- `match_queue` SQL migration (SQL in old WORKSPACE.md)
- Japanese library content stub
- Stripe paywall implementation
- Move rate limiter to Upstash Redis (in-memory breaks with Fluid Compute)
- CSP headers in next.config.ts
