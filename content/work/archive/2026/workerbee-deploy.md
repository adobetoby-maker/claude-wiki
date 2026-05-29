---
ai-first: true
type: reference
date: 2026-05-28
tags: [deploy, worker-bee, cloudflare, vercel, script, patterns]
status: active
---

# worker-bee.app Client Deploy Workflow

## For future Claude
One-command deploy for new client sites to `clientname.worker-bee.app`. Script at `~/deploy-client.sh`. Load this when deploying any new client site to a worker-bee subdomain. Credentials (Cloudflare token, Zone ID) are baked into the script — do not add them here.

---

## The Command
```bash
~/deploy-client.sh /Users/drive/project-folder clientname
```
One command: Vercel build + deploy → Cloudflare DNS A record → Vercel alias. ~60 seconds to a live URL.

## First-Time Setup (new project, one-time only)
1. `vercel link --cwd /path/to/project` — interactive, run once
2. `~/deploy-client.sh /path/to/project clientname`
3. Vercel dashboard → project → Settings → Deployment Protection → OFF

## Subsequent Deploys (already-linked projects)
Just run the script. No dashboard changes.

## Manual Steps (if script fails)
```bash
vercel --prod --cwd /path/to/project
# Add Cloudflare DNS: A record, name=clientname, ip=76.76.21.21, proxy=OFF
vercel alias set <deploy-url>.vercel.app clientname.worker-bee.app
```

## DNS Rules
- Vercel IP: `76.76.21.21`
- Proxy MUST be OFF (grey cloud) — orange proxy breaks Vercel SSL
- Zone: worker-bee.app on Cloudflare
- Account ID: `53c642cb4b27b3b66628e60a7efa0c27`

## Failure Mode
Cloudflare token returns 0 zones → was created without zone resource properly scoped. Fix: regenerate token at dash.cloudflare.com with explicit `worker-bee.app` zone selected.
