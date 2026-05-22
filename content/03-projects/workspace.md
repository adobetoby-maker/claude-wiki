# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Formatting Rules — Non-Negotiable

**Never wrap URLs, file paths, or shell commands in asterisks.**

- URLs: plain text only — `https://example.com` not `**https://example.com**`
- File paths: backtick or plain — `/Users/drive/project` not `**/Users/drive/project**`
- Commands: backtick only — `npm run dev` not `**npm run dev**`

Asterisks inside or around a URL or path break it when copied. No exceptions.

# VISUAL REVIEW — NON-NEGOTIABLE

Observable trigger: `git diff HEAD --name-only | grep -qE '\.(tsx|css|svg|png)'` exits 0 → STOP → screenshot + video BEFORE any "done" statement.

Two canonical failures: **(a) iter-16** — shader scored +0.25 from code intent, not pixels; every section regressed; iter-18 repair wiped the gain. **(b) iter-19** — element collisions near footer, harness scroll stopped short, single-viewport blindness at 1440.

**Required at every iteration:**
1. All 4 viewports: 375 mobile, 1440 desktop, 2560 4K, 2560@2x (5K). Missing = INCOMPLETE.
2. Scroll to `documentElement.scrollHeight - window.innerHeight`. Footer in final frame. Footer missing = harness broken.
3. Read all 4 viewport PNGs with Read tool — describe each section.
4. Extract frames: `ffmpeg -i review.webm -vf fps=2 frames/frame_%03d.png`. Read final 3 frames.
5. Two quoted observations per score dimension — one PNG, one video. Video is dominant for motion.
6. Any regression at any viewport → score drops. No exceptions.

Full protocol: `~/.claude/rules/visual-review-non-negotiable.md`

# Tool Capabilities — Quick Ref

| Tool | Pattern | When |
|---|---|---|
| `Bash(run_in_background=True)` | fire-and-forget, notified on complete | builds, long installs, dev servers |
| `Agent(subagent_type=X)` | delegate to specialist | code review, exploration, parallel work |
| Multiple `Agent(...)` in one message | run in parallel | independent tasks — always batch these |
| `CronCreate` | schedule recurring runs | daily jobs, cleanup, queue polling |
| `WebSearch` / `WebFetch` | live docs + research | before writing unfamiliar APIs |
| `Edit` over `Write` | sends diff only | always prefer for existing files |

# Autonomy Rules

**Act without asking permission** unless the action is irreversible or destructive.

**Just do it:** create/edit/rename files, push branches (non-main), run builds/lints/tests, start services (Vercel/Supabase/CF/MCPs), install packages, generate content, research.

**Pause and confirm first:** `rm`/`delete`/`drop`, force push to main, `git reset --hard`, drop DB tables, delete prod data, cancel deployed services.

When in doubt: branch and build. Never delete; always ask.

---

# Decision Defaults

| User says / context | Default action |
|---|---|
| Project name with no context | Run mem-search before answering |
| "like we did with X" or "remember when" | Run mem-search for X before guessing |
| "blog post" | Use `lib/articles.ts` in jrs-auto-repair — NOT markdown files |
| "add auth" to jrs or silver-creek | Ask: `/admin` (cookie) or `/portal` (Supabase JWT)? |
| "deploy" with no target | Check bindings: `cat wrangler.jsonc | grep -E "d1_databases|r2_buckets"` → none → Vercel |
| Mentions animation / scroll / parallax / hover | Use `record.js` (video) — NOT `screenshot.js` |
| New feature, no branch specified | Create `feature/<short-name>`, push, no confirmation needed |
| Mentions Twin Falls or Magic Valley | Context is jrs-auto-repair; use `lib/shopInfo.ts` |
| Mentions "blueprint" | Context is manage-worker-bee, `lib/blueprintStore.ts` |
| Mentions "vault" | Context is manage-worker-bee, `lib/vaultStore.ts` |
| Unknown term / project jargon | Check `~/.claude/vocabulary.md` before asking |
| Returns after a gap / clinic break | Run mem-search on recent projects before starting work |
| "build this website" or new site | Run `ls scores.md || echo MISSING` — MISSING → research first |

---

# Vocabulary

Full vocabulary: `~/.claude/vocabulary.md` — check before asking "what do you mean by X?"

- "Blueprint" = @xyflow/react canvas in manage-worker-bee — visual client site architecture
- "Vault" = AES-256-GCM credential store in manage-worker-bee
- "Portal" = customer area at `/portal` (Supabase JWT) — NOT the admin area
- "Admin" = internal ops at `/admin` (cookie auth) — NOT Supabase admin role
- "Articles" = `lib/articles.ts` TypeScript arrays — NOT markdown files
- "Dispatch" = silver-creek automated driver notifications (Vercel cron + CF Worker)
- "XP tier" = LinguaLens learner level (Beginner→Maestro) — separate from rank tier
- "Rank tier" = LinguaLens matchmaking rank (Bronze→Unreal) — separate from XP tier
- "Pilot" = climb-utah (first climbing site, Phase 1 validation)

---

# Failure Patterns

- Mixing `/admin` and `/portal` auth in jrs/silver-creek — cookie session and Supabase JWT overwrite each other silently
- Importing `lib/supabase/admin.ts` in any client component — service role key leaks to browser
- Running `npm run dev` without `-H 0.0.0.0` — Tailscale preview pane breaks, no error shown
- Creating markdown files for blog posts in jrs-auto-repair — blog reads from `lib/articles.ts` only
- Adding a LinguaLens tab without updating both `TabKey` in `app-state.tsx` AND `TAB_COMPONENTS` in `tab-registry.ts`
- Using `screenshot.js` to verify animations — screenshots freeze mid-animation; use `record.js`
- Calling `claude-flow` as `npx @claude-flow/cli@latest` — always use the global `claude-flow` binary
- Editing CLAUDE.md frequently — busts the preamble cache every session

---

# Memory Triggers — Run mem-search Proactively

- User mentions a project I haven't touched this session → search before answering
- User says "like we did with X" or "remember when..." → search for X before guessing
- User asks about a past decision or why something is built a certain way → search first
- User returns after a break or clinic shift → search recent work to re-orient

---

# Claude Bootstrap — Drive's Workspace

## Projects

| Project | Path | URL |
|---|---|---|
| Jr.'s Auto Repair | `/Users/drive/jrs-auto-repair` | jrsautorepair.worker-bee.app |
| manage.worker-bee | `/Users/drive/manage-worker-bee` | manage.worker-bee.app |
| LinguaLens | `/Users/drive/language-lens-elite` | language-lens-elite.worker-bee.app |
| Silver Creek Logistics | `/Users/drive/silver-creek-logistics` | silvercreeklogistics.worker-bee.app |
| Orthobiologic Pathways | `/Users/drive/orthobiologic-pathways` | orthobiologicpathways.com |
| Toby Anderton MD | `/Users/drive/tobyandertonmd` | tobyandertonmd.vercel.app |
| Climb France | `/Users/drive/climb-france` | climb-france.vercel.app |
| Climb Brasil | `/Users/drive/climb-brasil` | climbbrasil.com |

> Next.js 16 + React 19: read `node_modules/next/dist/docs/` before writing Next.js code — breaking changes from training data.

## Commands

| Project | Dev | Build | Test |
|---|---|---|---|
| jrs-auto-repair | `npm run dev` (port 3000) | `npm run build` | `npm run test` (Vitest) |
| manage-worker-bee | `npm run dev` (port 3000) | `npm run build` | — |
| language-lens-elite | `npm run dev` (Vite/CF) | `npm run build` | `npm run lint` |
| silver-creek-logistics | `npm run dev` (port 3000) | `npm run build` | `npm run lint` |
| orthobiologic-pathways | `npm run dev` (port 3000) | `npm run build` | — |
| tobyandertonmd | `npm run dev` (port 3000) | `npm run build` | — |
| climb-france | `npm run dev` (port 3000) | `npm run build` | — |

Always start with `-H 0.0.0.0` flag for Tailscale access.

## Before Touching Each Project

- **jrs-auto-repair**: read `lib/shopInfo.ts` + `lib/supabase/` (3 clients). See `jrs-auto-repair/CLAUDE.md`.
- **manage-worker-bee**: read `lib/blueprintStore.ts` + `lib/vaultStore.ts`. See `manage-worker-bee/CLAUDE.md`.
- **language-lens-elite**: read `src/components/tab-registry.ts` + `src/state/app-state.tsx`. See `language-lens-elite/CLAUDE.md`.
- **silver-creek-logistics**: read `lib/shopInfo.ts` + `lib/drivers.ts`. Same dual-auth as jrs.
- **orthobiologic-pathways**: R3F + Framer Motion, no Supabase, no auth. Use `record.js` for all visual changes.
- **tobyandertonmd**: no Supabase, no auth, no tests. Content in `src/components/`.
- **climb-france**: Next.js 16 + Vercel. 3-language i18n. See `climb-france/CLAUDE.md`.

## Dev Tools — Split View

| What | Command |
|---|---|
| Start devtools | `node /Users/drive/devtools/server.mjs` |
| Local URL | http://localhost:3333 |
| Tailscale URL | http://100.117.143.57:3333 |
| Screenshot (static) | `node ~/screenshot.js <port> 0,540,1080` |
| Video (animations) | `node ~/record.js <port>` |
| Mobile video | `node ~/record.js <port> --mobile` |
| Frame extract | `ffmpeg -i review.webm -vf fps=2 frames/frame_%03d.png` |

node-pty fix (if `posix_spawnp failed`): `cd /Users/drive/devtools && npm rebuild node-pty`

## Memory + Bootstrap

- **Flat-file memory**: `~/.claude/projects/-Users-drive/memory/`
- **RAG**: `claude-flow memory search --namespace claude-memories -q "query"`
- **Always use `claude-flow` global binary** — NEVER `npx @claude-flow/cli@latest`
- Session context injects automatically via `~/.claude/hooks.json` + `~/.claude/bootstrap/session-start.sh`

## Blueprint Progress Reporting

After significant work on any tracked project:

```bash
curl -s -X POST https://manage.worker-bee.app/api/blueprints/update \
  -H "x-api-key: 9fd6a40a79137d7fdb4ea7dc97d7c40478af2fae339dc8b25cc4595bd8dd1747" \
  -H "content-type: application/json" \
  -d '{"siteId": "<UUID>", "nodes": [...], "edges": [...], "summary": "..."}'
```

Resolve site name → UUID via `supabaseAdmin.from('sites').select('id,name')` from manage-worker-bee.
