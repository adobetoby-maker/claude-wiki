---
ai-first: true
type: project
date: 2026-05-28
tags: [manage-worker-bee, agency, blueprint, pipeline]
status: active
---

# manage-worker-bee

## For future Claude
The agency operating system. Always check this before working on any WB pipeline or multi-site task.

---

## Identity
- **Path:** `/Users/drive/manage-worker-bee`
- **URL:** https://manage.worker-bee.app (live, Vercel)
- **Stack:** Next.js 16 App Router, Supabase (`qnrkifdbkcbacgznoabs`), AES-256-GCM vault
- **Deploy:** `vercel --prod` (alias already wired)

## What It Does
Agency dashboard for the Worker-Bee build pipeline:
- Sites registry (56+ sites tracked)
- Blueprint canvas (`@xyflow/react`) — visual client site architecture
- Encrypted vault (AES-256-GCM credential store)
- Build Machine (7-phase pipeline with streaming terminal)
- Agents tab (XyFlow canvas showing 7 droid phases)
- WB Pipeline Runs tracker (`wb_pipeline_runs` Supabase table)
- Maintenance dispatch queue

## Key Files
- `lib/blueprintStore.ts` — blueprint canvas state
- `lib/vaultStore.ts` — AES-256-GCM credential vault
- `app/api/wb-run/route.ts` — GET/POST pipeline runs
- `app/api/blueprints/update/route.ts` — blueprint progress reporting
- `app/sites/[id]/build/page.tsx` — build machine per site

## Blueprint API
```bash
curl -X POST https://manage.worker-bee.app/api/blueprints/update \
  -H "x-api-key: 9fd6a40a79137d7fdb4ea7dc97d7c40478af2fae339dc8b25cc4595bd8dd1747" \
  -H "content-type: application/json" \
  -d '{"siteId": "<UUID>", "nodes": [...], "edges": [...], "summary": "..."}'
```

## Build Machine Pipeline
1. Blueprint canvas → AI Generate (wizard) → 6 identity nodes saved
2. Build page → site type + mode (New Build / Iteration) → generate spec
3. One-Click Build → POST to `build-api.worker-bee.app/run` (x-api-key: `wb-build-local-9f4a2c`)
4. Claude CLI: Phase 0 → 1 → 2 → 2.5 → 3 → 4 → 5

## Maintenance System
- `/maintenance` — Client Requests queue + Dispatch form
- `maintenance_requests` table: id, client_name, raw_request, status, dispatched_at
- `/request` — public client-facing intake form
- Dispatch: fetches CLAUDE.md via `/api/github/claude-md`, generates spec, fires build-api

## Failure Patterns
- Editing blueprint without migrating legacy `{nodes, edges}` flat format → data loss on save
- `localPath` derived from `github_repo.split('/')[1]` — "Dr." prefixes cause wrong dir
- GITHUB_TOKEN required in Vercel for maintenance dispatch (repo scope)
