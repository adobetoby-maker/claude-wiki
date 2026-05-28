---
ai-first: true
type: critical-facts
always-load: true
date: 2026-05-28
---

# Critical Facts — Always Loaded

## Identity
- **Operator:** Toby Anderton (Drive) — adobetoby@gmail.com
- **Tailscale IP:** 100.117.143.57
- **Machine:** Mac Studio M1 Ultra

## Platform Constants
- **Blueprint API:** https://manage.worker-bee.app — key: `9fd6a40a79137d7fdb4ea7dc97d7c40478af2fae339dc8b25cc4595bd8dd1747`
- **Vercel A record:** 76.76.21.21 (all marketing/affiliate sites)
- **worker-bee.app subdomains:** CF Workers + D1/R2/KV. Everything else: Vercel.

## Key URLs
- Workspace: https://manage.worker-bee.app
- TAC Deck: http://100.117.143.57:3099
- Chief of Staff: http://100.117.143.57:8094
- Build Studio: https://manage.worker-bee.app/build-studio
- Terminal: https://macs-mac-studio.tail94170b.ts.net:7681

## Agent Commands
- `jr "task"` → Hermes Jr (Max OAuth, synchronous, output returned to TAC)
- `wba "task"` → Worker Bee Agent (Max OAuth, inline)
- `wba -b "task"` → wba background queue
- `dispatch --bg "task"` → Hermes fire-and-forget
- `hermes --profile sitemanager -z "task"` → SiteManager (orthobiologic)

## Deploy Commands
- Marketing/affiliate: `vercel --prod` (CLI auth in ~/.config/vercel/auth.json)
- worker-bee subdomains: `wrangler deploy` (requires D1/R2/KV bindings)
- Verify live: `curl -sI <url> | head -1` → must be HTTP/2 200

## Supabase Projects
- manage-worker-bee: `qnrkifdbkcbacgznoabs`
- jrs-auto-repair: see [[work/active/jrs-auto-repair]]
- language-lens-elite: `pollhlkgltdkdskdzsgd`
- mountain-edge: `fddmjywlegzgwehsxzei`
