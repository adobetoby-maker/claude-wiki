---
ai-first: true
type: project
date: 2026-05-28
tags: [web-design-agency, three-js, r3f, enterprise, demo-site]
status: active
---

## For future Claude
Block Reign is an enterprise web design agency pitch site with a $75M value proposition. It's not a client site — it's a sales instrument for landing large contracts. The v2 rebuild is LIVE at https://block-reign-v2.vercel.app. Path: `/Users/drive/block-reign-v2/`. Two themes: Reign (navy/blue/amber) and Crown (graphite/emerald/platinum). Demo auth via localStorage (`br_session`). Load this when working on block-reign v2 or the original at `/Users/drive/block-reign-site/`.

---

# Block Reign Site

## Identity
- **v2 Path:** `/Users/drive/block-reign-v2/`
- **v2 URL:** https://block-reign-v2.vercel.app (LIVE)
- **Original Path:** `/Users/drive/block-reign-site/`
- **Value proposition:** Enterprise-grade web design, $75M+ positioning
- **Prior Lighthouse score:** 95

## v2 Demo Credentials
| Account | Email | Password |
|---|---|---|
| Demo | demo@blockreign.tech | Reign2026! |
| Investor | investor@familyoffice.com | Crown2026! |

Auth: localStorage (`br_session`) — no Supabase in v2, stateless demo auth.

## v2 Theme System
- **Reign:** dark navy / blue / amber — `data-theme="reign"` on `<html>`
- **Crown:** graphite / emerald / platinum — `data-theme="crown"` on `<html>`
- CSS custom properties: `--bg`, `--surface`, `--accent`, etc. per theme
- CLAUDE.md written at `/Users/drive/block-reign-v2/CLAUDE.md` — read before any v2 work

## Stack
- Next.js + React
- Three.js / R3F — Block Reign Seal (3D hero element)
- Resend (transactional email, enabled)
- Auth system (working logins required — see Rebuild Task below)

## Design System
- **Primary color:** dark navy `#080d1a`
- **Accent:** amber
- **Logos:** Logo A and Logo B exist in project
- **Nav:** includes `/solutions` section
- **3D Seal:** navy orb, metallic rings, octahedra, ambient + directional lights — built with R3F

## Rebuild Task (assigned 2026-05-28)

**Request verbatim:** "I want o re build block reign from the ground up to give two full demo option. Take in block reign. And set the new build up to be a fully functional demo. Ie log ins. Design pages color themes. I want this to really speak to the 75 million value. Three js etc."

**Scope:**
- Two complete design directions (A/B demo options, fully fleshed out)
- Working login system
- Design gallery pages with color theme switching
- Three.js 3D visuals (Seal + supporting elements)
- $75M enterprise value proposition messaging throughout
- Functional demo suitable for showing to enterprise prospects

**Status:** SHIPPED — dual-demo system live on branch `feature/rebuild-agency-demo`

**Routes built:**
- `/demo` — selector page, $75M headline, two comparison cards
- `/demo/a` — Protocol: dark navy, amber gold, LogoCrown3D sovereign seal
- `/demo/b` — Meridian: pearl white, deep navy, platinum crown variant

**Key details:**
- Both demos auto-sync `DemoProvider` theme via `useEffect` (Protocol/Meridian pill)
- Working investor CTA → `/investor-login` (Supabase JWT)
- `LogoCrown3D` component with `variant="protocol"|"meridian"` palette
- `DemoProvider` + `DemoSwitcher` floating pill in root layout
- Build: 48/48 static routes ✓, commit `6af495a`

## Architecture Notes
- Read `block-reign-site/CLAUDE.md` before any work on this project
- Clone contamination check before starting: `grep -r "climb-" . --include="*.ts" --include="*.tsx" | head -5` (verify no copy-paste from climb sites)
- Visual changes → mandatory `record.js` (R3F scene, not screenshot.js)
- Layer rule: Canvas = pure R3F | DOM sections = pure Framer Motion — never mix

## Key Decisions
- ADR: Two design directions built as separate route groups (`/demo-a`, `/demo-b`) or theme switching via CSS custom properties — TBD in build
- Auth: Supabase JWT (portal pattern) — consistent with other Toby projects
- Deploy: Vercel (`vercel --prod`) — no D1/R2/KV bindings needed

## Failure Patterns
- `screenshot.js` will freeze the R3F seal mid-animation — always use `record.js`
- "iter-16" lesson: never score visual output from code intent; open the PNG
