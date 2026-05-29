---
ai-first: true
type: project
date: 2026-05-28
tags: [orthobiologic, r3f, three-fiber, framer-motion, static, no-supabase]
status: active
---

# Orthobiologic Pathways

## For future Claude
Medical/biologic patient info site for orthopedic regenerative medicine. R3F + Framer Motion, NO Supabase, NO auth, NO lib/ directory. SiteManager agent handles this. Load when working on orthobiologicpathways.com or LBS Pro orders.

---

## Project
- **Path:** `/Users/drive/orthobiologic-pathways`
- **URL:** orthobiologicpathways.com (Vercel)
- **Deploy:** `vercel --prod`
- **Dev:** `npm run dev` (port 3000, `-H 0.0.0.0`)
- **Agent:** `hermes --profile sitemanager -z "task"` handles this project

## Stack — Key Deviations
- **React Three Fiber** (`@react-three/fiber` + `@react-three/drei`) — 3D hero sections
- **Framer Motion** — page animations; combined with R3F creates stacking context complexity
- **NO Supabase** — fully static, no database, no auth of any kind
- **NO lib/ directory** — data and types live in `components/` or `app/` directly
- **NO test script** — `npm run test` does not exist
- **Flat routes** — no `(site)/` grouping; all routes at root level

## Critical: Visual Verification
ALL visual changes require video — screenshots freeze R3F scenes mid-render (ADR-0003):
```bash
node ~/record.js <port>           # required for ANY R3F or Framer Motion change
node ~/record.js <port> --mobile  # R3F behaves differently on mobile
ffmpeg -i review.webm -vf fps=2 frames/frame%03d.png
```
Never use `screenshot.js` alone for this project.

## Finding Data
No lib/ directory. When looking for content arrays, types, or data:
```bash
grep -r "const.*=" components/ app/ --include="*.ts" --include="*.tsx"
```
Data lives in `components/` or directly in `app/[route]/page.tsx`.

## Routes (flat)
`/` `/peptides` `/peptide-library` `/peptide-stacking` `/stem-cells` `/stem-cells-prp` `/non-surgical-peptides` `/non-surgical-uses` `/surgery-recovery` `/stacking` `/consultation` `/consultation-flow` `/shop` `/shop-gate` `/patient-dashboard` `/dashboard` `/auth` `/auth-sign-in`

## Vocabulary
- "Patient dashboard" = `/patient-dashboard` — static/demo, no real auth
- "Shop gate" = `/shop-gate` — access control page, no real e-commerce
- "Consultation flow" = `/consultation-flow` — multi-step form, no backend

## Failure Patterns
- Looking for `lib/` directory → doesn't exist; data is in `components/` or `app/`
- Using `screenshot.js` for R3F/Framer Motion → frozen frame, false-positive score (ADR-0003)
- Running `npm run test` → command not found
- Adding Supabase or auth without explicit ask — not in this stack
