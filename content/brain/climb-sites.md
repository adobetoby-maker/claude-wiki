---
ai-first: true
type: reference
date: 2026-05-28
tags: [climb, affiliate, nextjs, cloudflare, vercel, i18n]
---

# Climb Sites — Reference

## For future Claude
Six affiliate climbing guide sites. All share the same dark-premium affiliate pattern but deploy differently and use different language keys. Load this when working on any climb-* project. Critical: never mix country facts, language keys, or emergency numbers between sites.

---

## Site Directory

| Site | Path | URL | Deploy | Stack | Lang keys |
|---|---|---|---|---|---|
| climb-brasil | `/Users/drive/climb-brasil` | climb-brasil.worker-bee.app | `wrangler deploy` | Next.js + @opennextjs/cloudflare | `en` \| `pt` \| `es` |
| climb-france | `/Users/drive/climb-france` | climb-france.vercel.app | `vercel --prod` | Next.js 16 | `en` \| `fr` \| `de` |
| climb-spain | `/Users/drive/climb-spain` | climb-spain.worker-bee.app | `opennextjs-cloudflare build && opennextjs-cloudflare deploy` | Next.js + @opennextjs/cloudflare | varies |
| climb-utah | `/Users/drive/climb-utah` | climb-utah.worker-bee.app | `vercel --prod` | Next.js | English only |
| climb-kalymnos | `/Users/drive/climb-kalymnos` | climb-kalymnos.worker-bee.app | `opennextjs-cloudflare build && opennextjs-cloudflare deploy` | Next.js + @opennextjs/cloudflare | `en` \| `de` \| `it` \| `el` |
| climb-idaho | `/Users/drive/climb-idaho` | climb-idaho.vercel.app | `vercel --prod` | Next.js | English only |

## Dev Ports
- climb-france: always port 3002 (`npm run dev -- -H 0.0.0.0 -p 3002`)
- all others: port 3000 default

## Critical: No Cross-Site Contamination
All sites were cloned from each other. Before ANY work run the clone-check:
```bash
# In climb-france: look for Kalymnos/Brasil/Greece contamination
grep -r "Kalymnos\|Brasil\|Greece\|EKAB\|166\|Kos" src/ --include="*.ts" --include="*.tsx"
# In climb-kalymnos: look for France/Brasil contamination
grep -r "France\|Brasil\|SAMU\|15 \|17 \|18 " src/ --include="*.ts" --include="*.tsx"
```

## Country-Specific Facts (never hardcode — read from SITE.md in each project)

| Site | Country | Emergency | Accent color | Currency |
|---|---|---|---|---|
| climb-brasil | Brazil | 192 SAMU / 190 police | TBD | BRL |
| climb-france | France | 15 SAMU / 17 police / 18 fire | #D4185A French red | EUR |
| climb-kalymnos | Greece | 166 EKAB / 100 police / 108 coast guard | #1B74C8 Aegean blue | EUR |
| climb-utah | USA | 911 | TBD | USD |
| climb-idaho | USA | 911 | dark gold #C9A84C | USD |

## Affiliate Tags
- climb-idaho: Amazon tag `climbidaho-20`, REI, Booking.com, WorldNomads, GetYourGuide, Language Threshold
- climb-brasil: Amazon tag `climbing00bb-20` (shared with medicalspanish/constructionspanish)

## TypeScript Rule (i18n sites)
After ANY Lang type change: `npx tsc --noEmit`
`Record<Lang, string>` requires all keys — TypeScript catches missing ones, but only if you run it.

## Shared Stack Facts
- All: dark premium UI, affiliate-first data layer, static generation
- No Supabase, no auth, no database on any climb site
- Images: Pexels (free commercial license, no attribution)
- Pattern source: climb-brasil is the original; others are clones

## Failure Patterns
- Cloning without running clone-check → wrong country facts ship silently (20+ errors found in Brasil→Kalymnos clone)
- Using wrong deploy command → CF Worker sites need `wrangler deploy` / opennextjs, not `vercel --prod`
- Wrong lang keys in TypeScript → missing Record key fails at runtime, not build time if type is loose
- climb-france dev on wrong port → screenshot/record scripts point to wrong port
