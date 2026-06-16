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

## Community Vault Tools (2026-05-28)
**Decision:** Adopt obsidian-mind AI-first format + obsidian-second-brain 34-command research toolkit.
**Tools adopted:**
- obsidian-mind (breferrari) — vault structure template, tiered SessionStart loading (~2K tokens budget)
- kepano/obsidian-skills — Agent Skills SKILL.md format, 5 base skills (obsidian-markdown, obsidian-bases, etc.)
- obsidian-second-brain (eugeniughelbur) — 34 slash commands, research toolkit, self-updating vault, PostCompact hook
**Installed at:** `~/.claude/skills/obsidian-second-brain/`, commands in `~/.claude/commands/`, vault path in `~/.claude/settings.json`
**Alternative rejected:** Staying with flat `~/.claude/projects/-Users-drive/memory/` — no community leverage, no visual navigation.

## claude-wiki CI/CD: GitHub Actions + Vercel Classic Token (ADR-0009)
**Decision:** Auto-deploy uses GitHub Actions workflow (`push: v4 → vercel --prod`) with a Vercel classic personal token, NOT the OAuth token from Vercel CLI login.
**Date:** 2026-05-28
**Rationale:** Vercel OAuth login (`vercel login`) creates an OAuth token with scope `openid offline_access` only — cannot be used to create classic tokens or make project modification API calls. Classic personal token is required for CI automation. Fork: `adobetoby-maker/claude-wiki`. Upstream (Quartz updates): `jackyzha0/quartz.git`.
**Alternative rejected:** Using Hermes MCP token for Vercel API calls — scope too limited; wrangler pattern (not applicable to marketing/docs sites per ADR-0003).
**Secrets required:** `VERCEL_TOKEN` (classic personal), `VERCEL_ORG_ID` = `team_WPMJl6w7aYPU9xP3sr8Xx3uN`, `VERCEL_PROJECT_ID` = `prj_pBIOfQISvrNzw36x6HRiRCUFPhv5`.
**Rule:** If Vercel deploy fails in CI, first check token type — OAuth tokens fail silently on project API calls.

## SessionStart Bootstrap: Python Frontmatter Stripping (ADR-0010)
**Decision:** `session-start.sh` uses Python `re.sub` to strip only the FIRST frontmatter block, not `sed '/^---$/,/^---$/d'`.
**Date:** 2026-05-28
**Rationale:** The sed pattern treats ANY `---` line as a frontmatter delimiter. North Star.md has a `---` horizontal rule after `## For future Claude` — sed was silently deleting Mission, Three Pillars, and Active Priorities from session context. Python `re.sub(r'^---\n.*?\n---\n?', '', content, count=1, flags=re.DOTALL)` with `count=1` only removes the first block.
**Downstream fix:** Also added `extract_preamble()` function to pull `## For future Claude` one-liners from each active project file — gives Tier 3 project context without reading full files.

## tac-brain: Public GitHub Repo for Sharing /tac (2026-05-29)
**Decision:** Share the full TAC personal assistant structure via a dedicated public GitHub repo at https://github.com/adobetoby-maker/tac-brain — NOT by exposing a URL or copying individual skill files.
**Date:** 2026-05-29
**Scope:** 57 files, 7,800+ insertions — SOUL.md, AGENTS.md, vocabulary.md, rules (12), categories (8), personalities (4), hooks (5), bootstrap (2), memory (15 safe files), plugins (2), sanitized MCPs (2).
**Security incident:** GitHub Push Protection blocked the initial push and caught 3 hardcoded secrets: CF zone DNS token in `memory/project_manage_worker_bee.md`, Google OAuth Client ID and Client Secret in `AGENTS.md`. All three were redacted with `sed -i ''` before push.
**Alternative rejected:** Sharing via URL (no shareable URL exists for Claude Code skills) or exporting individual files (incomplete, loses the interconnected structure).
**Rule:** Before pushing any memory or config repo, grep for tokens/credentials: `grep -rE "(cfut_|GOCSPX-|sk-|Bearer )" . | grep -v node_modules`.

## Harness Bid: Cherry-Pick Phase 6 Only (2026-05-29)
**Decision:** From "Jay's" bid for an LLM-agnostic harness (7 phases), adopt only Phase 6 — local model benchmark matrix.
**Date:** 2026-05-29
**Rationale:** Phases 1–5 replicate work already done in TAC + obsidian-second-brain + agent-ecosystem. Phase 7 (monetization) is premature. Phase 6 (benchmark matrix comparing Sonnet vs local models on real TAC tasks) has unique value not covered by current stack.
**Alternative rejected:** Full harness build — too much overlap with existing system, significant rebuild cost with low incremental gain.

## Neural Map Graph: Obsidian Graph View Over Custom Three.js (2026-05-29)
**Decision:** Use Obsidian's actual graph view (or `react-force-graph`, which Obsidian uses internally) for the neural map 3D visualization — NOT the custom Three.js force-directed implementation in `NeuralGraph3D.tsx`.
**Date:** 2026-05-29
**Context:** `NeuralGraph3D.tsx` was written with custom `useFrame` physics (28 nodes, 45+ edges, emissive spheres) but NOT integrated. User redirected mid-build with "lets just use obsidians graph view."
**Options to evaluate:** (1) embed published vault graph from claude-wiki-two.vercel.app as iframe, (2) use `react-force-graph` (the library Obsidian uses), (3) link out to Obsidian's own graph view.
**Simplest path:** `react-force-graph` — gives Obsidian aesthetic with minimal rework from existing `NeuralGraph3D.tsx` data model (node/edge arrays already defined).
**Status:** PENDING — `NeuralGraph3D.tsx` exists at manage-worker-bee but is not yet integrated.

## Vercel SPA Routing: `routes` + `handle: filesystem` for API Functions (ADR-0014)
**Decision:** For non-Next.js SPAs on Vercel with `api/` serverless functions, use the deprecated `routes` array with `{ "handle": "filesystem" }` sentinel — NOT `rewrites`.
**Date:** 2026-05-29
**Rationale:** `rewrites` are evaluated BEFORE Vercel Function routing. A catch-all `/(.*) → /index.html` rewrite intercepts `/api/*` paths and returns SPA HTML, making the function unreachable. The `routes` array with `{ "handle": "filesystem" }` as first entry tells Vercel's edge to check static files and serverless functions before applying the SPA catch-all.
**Current deployed state** (`constructionspanish-app/vercel.json` and `medicalspanish-app/vercel.json`):
```json
{
  "routes": [
    { "handle": "filesystem" },
    { "src": "/(.*)", "dest": "/index.html" }
  ]
}
```
**Note:** An intermediate form with explicit `{ "src": "/api/(.*)", "dest": "/api/$1" }` also works but the `handle: filesystem` form is cleaner — auto-discovers all `api/` functions without listing them.
**Files affected:** Both updated 2026-05-29.
**Alternative rejected:** Removing only the `/api/(.*)` rewrite entry — does not fix the issue; the `/(.*) catch-all` still intercepts API paths when `rewrites` is the mechanism.
**Rule:** Any Vite/React SPA on Vercel that has `api/` functions → use `routes` array with `handle: filesystem`, not `rewrites`.

## HTML5 Over React for Simple Affiliate/SEO Sites (ADR-0015)
**Decision:** Simple affiliate/SEO/marketing sites → HTML5 + vanilla JS (+ Three.js via importmap if 3D needed). Auth/dynamic/SaaS sites → Next.js/React.
**Date:** 2026-05-29
**Decision matrix:**
| Site type | Stack |
|---|---|
| Affiliate, SEO articles, lead gen, no auth | HTML5 + vanilla JS |
| 3D hero section wanted | + Three.js via `<script type="importmap">` |
| Auth, dashboard, Supabase, forms with state | Next.js + React |
| worker-bee.app subdomain with D1/KV/R2 | Cloudflare Workers |
**Rationale:** HTML pages load faster, have no build step, are simpler to debug, and look "beautiful" (user's words). React adds complexity with no benefit for static/SEO-first sites. The bundler eliminates itself — Three.js loads via CDN importmap, email capture is a vanilla `fetch()` POST.
**First application:** `constructionspanish-app/index.html` rebuilt as standalone HTML5 page (2026-05-29). React components in `src/` preserved but not loaded.
**Alternative rejected:** Continuing with Vite/React for all sites — unnecessary complexity for content-only pages.

## Amazon Affiliate Tag Standardization (ADR-0016)
**Decision:** Use tag `climbing00bb-20` on all Drive sites as the placeholder affiliate tag until each niche gets its own Associates program tag.
**Date:** 2026-05-30
**Rationale:** Launching sites with a working affiliate tag is better than waiting for niche-specific tags. Revenue is low per site; consolidation under one tag is acceptable short-term. climb-brasil had `CLIMBBRASIL-20` (wrong) — corrected to `climbing00bb-20`.
**FTC requirement:** Every page with affiliate links must include: "As an Amazon Associate I earn from qualifying purchases."
**Links format:** `https://www.amazon.com/s?k=KEYWORDS&tag=climbing00bb-20` with `rel="noopener noreferrer sponsored"`.
**Future state:** Apply for Associates tags per niche (construction, medical, language-learning) and update per-site once approved.
**Sites using this tag:** constructionspanish-app, climb-brasil, and all 8 sites targeted by 2026-05-30 swarm.
**Alternative rejected:** Per-site tags from day one — requires separate Associates applications per niche; delays monetization.

## SRI Exception: gtag.js (2026-05-28)
**Decision:** Google's `gtag.js` script does NOT get an `integrity` attribute. No SRI hash.
**Rationale:** Google dynamically updates gtag.js. Any fixed SRI hash breaks the script loading silently. This is a known exception to the general "add integrity to external scripts" rule.
**Applies to:** All language-threshold sites using GA4 tag `G-RP0TZ1MP7E`, and any site using Google Analytics.
**Rule:** Security hook (PostToolUse:Write) will flag this — it is a correct exception, not a bug.
