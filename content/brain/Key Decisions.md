---
ai-first: true
type: decisions
date: 2026-05-28
tags: [architecture, decisions, ADRs]
---

# Key Decisions

## For future Claude
Architectural decisions that must not be reversed without explicit discussion. Each entry has a date, rationale, and the alternative that was rejected.

---

## Platform Routing (ADR-0003)
**Decision:** Marketing/affiliate/portfolio sites → Vercel. worker-bee.app subdomains → Cloudflare Workers.
**Date:** 2026-05-22
**Rationale:** Wrangler auth expires and cannot be refreshed non-interactively. Vercel CLI auth persists in `~/.config/vercel/auth.json` across sessions. The moment any marketing site needed a deploy and wrangler was unavailable, the project stalled.
**Alternative rejected:** All sites on Cloudflare Pages — wrangler auth wall kills autonomous deploys.
**Rule:** Before writing `wrangler.jsonc` for any new project, confirm it has D1/R2/KV/cron needs. If not → Vercel.

## Auth Separation (ADR-0006)
**Decision:** `/admin` uses cookie-based auth. `/portal` uses Supabase JWT. Never mix.
**Rationale:** Mixing causes silent session overwrites in production. Cookie session and Supabase JWT overwrite each other with no error shown.
**Projects affected:** [[work/active/jrs-auto-repair]], [[work/active/silver-creek]]

## Blog Content in TypeScript (ADR-0004)
**Decision:** Blog/article content lives in `lib/articles.ts` TypeScript arrays, not markdown files.
**Projects affected:** jrs-auto-repair
**Rationale:** Markdown files are silently ignored by the blog reader. All content edits must go through articles.ts.

## Haiku for Mechanical Tasks (ADR-0005)
**Decision:** File rename, git commit, npm install, image resize, curl → Haiku. TypeScript architecture, debugging, content writing → Sonnet.
**Rationale:** Haiku is ~1/10 the cost of Sonnet. Mechanical tasks don't benefit from Sonnet reasoning.

## Memory System: Vault-First (2026-05-28)
**Decision:** Migrate from flat `~/.claude/projects/-Users-drive/memory/` to Obsidian vault at `/Users/drive/claude-wiki/content/`.
**Rationale:** Community resources (obsidian-mind, obsidian-second-brain) will improve the system. Vault gives visual navigation via wikilinks and graph view. AI-first format (frontmatter + wikilinks + recency markers) makes future-Claude retrieval more accurate.
**Alternative rejected:** Continue with flat files — loses community leverage, no visual navigation.

## Visual Review: Video Required for Animations (2026-05-17)
**Decision:** Any animated UI (Framer Motion, R3F, scroll-driven) requires `record.js`, not `screenshot.js`.
**Rationale:** Screenshots freeze mid-animation. iter-16 (Block Reign) gave a false +0.25 score from code intent — every section actually regressed. Cost: 2 wasted iterations.

## WB Pipeline: Observable Handoffs
**Decision:** Each pipeline droid produces a file artifact as its output contract (research-brief.json, scores.md, qa-report.md). Next droid reads the file, not a human's word.
**Rationale:** Auditable pipeline state from filesystem. No guessing whether a phase is complete.
