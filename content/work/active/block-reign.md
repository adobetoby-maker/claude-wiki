---
ai-first: true
type: project
date: 2026-05-28
tags: [block-reign, enterprise, threejs, r3f, nextjs, design-agency, demo]
status: active
---

# Block Reign

## For future Claude
Enterprise web design agency pitch site targeting the $75M value proposition. Full ground-up rebuild in progress as of 2026-05-28. Two complete demo options with working logins, design pages, color themes, and Three.js 3D visuals. Load this when working on block-reign-site, the rebuild, or any enterprise demo/pitch work.

---

## Project

| Version | Path | URL | Status |
|---|---|---|---|
| v2 (rebuild) | `/Users/drive/block-reign-v2` | https://block-reign-v2.vercel.app | LIVE |
| Original | `/Users/drive/block-reign-site` | (prior Vercel preview) | archived |

- **Deploy:** `vercel --prod` (from project dir)
- **Dev:** `npm run dev` (port 3000, `-H 0.0.0.0` for Tailscale)

## Current State (as of 2026-05-29)

**v2 SHIPPED.** Full ground-up rebuild live at https://block-reign-v2.vercel.app.

### v2 Demo Credentials
- `demo@blockreign.tech` / `Reign2026!`
- `investor@familyoffice.com` / `Crown2026!`

### v2 Theme System
Auth via localStorage `br_session` — stateless demo, no Supabase. Two themes toggled via `data-theme` attribute:
- **Reign:** dark navy / blue / amber
- **Crown:** graphite / emerald / platinum

### From the existing site (pre-rebuild)
- Colors: dark navy (`#080d1a`) + amber accent
- Three.js/R3F "Block Reign Seal": navy orb, metallic rings, octahedra, lights
- Logos A and B exist in the repo
- Demo email: `demo@blockreign.tech`
- Resend email enabled
- `/solutions` nav section
- Previously scored LH 95

### Rebuild Spec
User's exact request: "re build block reign from the ground up to give two full demo option. Take in block reign. And set the new build up to be a fully functional demo. Ie log ins. Design pages color themes. I want this to really speak to the 75 million value. Three js etc."

**Required:**
1. Two complete design directions (full demo options, switchable)
2. Working login system
3. Design pages + color themes per demo
4. Three.js 3D visuals (carry forward the Seal + enhance)
5. $75M enterprise value proposition messaging throughout
6. Speaks to high-end agency positioning

## Visual Review Protocol

See `[[03-projects/block-reign]]` for the canonical visual review protocol (iter-16 and iter-19 failure modes documented).

**Mandatory for all visual iterations:**
- `record.js` (not `screenshot.js`) — R3F + Framer Motion present
- All 4 viewports: 375, 1440, 2560, 2560@2x
- Scroll to `documentElement.scrollHeight - window.innerHeight`
- Two quoted observations per score dimension (PNG + video)

## Key Decisions

- **Why rebuild**: Demonstrate two full design directions to enterprise prospects; existing site is a single-direction demo
- **Three.js mandatory**: The Seal is a brand differentiator — carry forward and enhance
- **Working auth**: Demo credibility requires real login, not a placeholder
- **record.js always**: R3F/Framer Motion means screenshots miss motion quality

## Failure Patterns

- iter-16: GLSL shader scored +0.25 from code intent, not pixels — compliance section destroyed, mobile broken. Repair at iter-18 wiped the gain.
- iter-19: Element collisions near footer, harness scroll stopped short, single-viewport blindness at 1440.
- **New build risk**: Dual-demo switching — ensure theme context doesn't leak between demo A/B routes

## Environment

- `block-reign-site/.env.local` — Resend API key, auth secrets (check credential-map.md for retrieval)
- No D1/R2/KV → Vercel deploy (not wrangler)
