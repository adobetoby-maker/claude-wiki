---
ai-first: true
type: project
date: 2026-05-28
tags: [client-site, landing-page, nextjs, cja, crypto, archived]
status: archived
---

# Salvorias — CJA Web Services SAV Offer

## For future Claude
Landing page for CJA Web Services' 25-spot offer: SAV token holders get a custom WordPress site for 1,995 USD in SAV. Next.js 15, dark navy + amber. Build verified, deploy pending (needs manual `~/deploy-client.sh` run). Load this if CJA or Salvorias comes up.

---

## What It Is
Single-page landing: SAV token holders apply for a custom site. 8-field application form.

- **Contact:** jay@cjawebservices.com / info@cjawebservices.com
- **Path:** `worker-bee/salvorias/` (inside adobetoby-maker/worker-bee repo, branch `claude/implement-tac-wjgKS`)
- **Target:** `salvorias.worker-bee.app`

## Status (2026-05-16)
- Next.js 15 build passes (103 kB first load) ✅
- Deployed to Vercel ✅
- worker-bee.app DNS alias: pending (run `~/deploy-client.sh`)

## Design System
| Token | Value |
|---|---|
| Background | `#080d1a` deep navy |
| Primary accent | `#f59e0b` amber-500 (SAV gold) |
| Card surface | `#111827` / `#1f2937` |

Custom globals.css: `text-gradient-gold`, `glow-gold`, `glow-gold-sm`, `card-hover`

## 8 Sections
Navbar → Hero → SocialProof → Features (6-card grid) → Pricing → HowItWorks → ApplicationForm → Footer

## Form Wiring (incomplete)
Currently: 1.2s fake delay → success state. When resuming: Formspree (Option A), Supabase table (Option B), or Resend server action (Option C).

## Deploy
```bash
git fetch origin claude/implement-tac-wjgKS
git checkout claude/implement-tac-wjgKS
~/deploy-client.sh $(pwd)/salvorias salvorias
```
Then: Vercel dashboard → project → Settings → Deployment Protection → OFF.
