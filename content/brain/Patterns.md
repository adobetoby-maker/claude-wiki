---
ai-first: true
type: patterns
date: 2026-05-28
tags: [patterns, architecture, recurring]
---

# Patterns

## For future Claude
Recurring solutions that work. When you see a problem that matches a pattern here, use the pattern — don't reinvent.

---

## The 5-Method Rule (Deploy Failures)
When any deploy step exits non-zero, run all five in order before reporting failure:
1. `vercel --prod` — CLI cached auth in `~/.config/vercel/auth.json`
2. `env | grep -iE "cloudflare|vercel"` — token already in env
3. `gh repo view --json homepageUrl` — GitHub auto-deploy already wired
4. `mcp__claude_ai_Vercel__deploy_to_vercel` — Vercel MCP
5. `.github/workflows/deploy.yml` with VERCEL_TOKEN secret

Report what worked. Never report failure until all 5 are exhausted.

## Tiered Agent Dispatch
```
TAC (this session) → complex architecture, TypeScript, multi-file debugging
Jr (jr "task") → synchronous background tasks where TAC needs the output
wba -b ("task") → true fire-and-forget, TAC doesn't need results
dispatch --bg → Hermes-style queued work
```
Key invariant: `jr "task"` must use `Bash(timeout=600000)` — synchronous. Never `run_in_background=True` when TAC needs the output.

## SessionStart Tiered Loading
Load in order, stop when context budget fills:
1. CRITICAL_FACTS.md — always, ~120 tokens
2. brain/North Star.md — excerpt, ~200 tokens
3. work/active/ — filenames only, ~100 tokens
4. Recent memory (now.md / today-*.md) — ~400 tokens
5. Full project files — on-demand only when working on that project

## AI-First Note Writing
Every vault note written for future-Claude retrieval:
```yaml
---
ai-first: true
type: [concept|project|decision|pattern|person]
date: YYYY-MM-DD
tags: [relevant, tags]
---

## For future Claude
[what this note is, why it matters, when to load it]
```
- Use `[[wikilinks]]` for all referenced projects, people, decisions
- Recency markers: "(as of 2026-05, source.com)"
- Confidence levels where uncertain

## Droid Pipeline Pattern
Each droid: one phase, narrow expertise, observable handoff.
```
Input: file artifact from predecessor
Work: concrete numbered steps with bash commands
Output: file artifact that triggers successor
Gate: refuse to proceed if input artifact is missing
```
See: [[brain/how-to-build-droids]]

## WB Site Build Sequence
1. `python3 ~/.claude/skills/ui-ux-pro-max/scripts/design_system.py` → MASTER.md
2. Blueprint at manage.worker-bee.app → topological card order
3. Build cards in dependency order — deps first
4. `npx tsc --noEmit` → zero errors before visual QA
5. `node ~/screenshot.js <port> 0,540,1080` → score 6 dimensions
6. `node ~/record.js <port>` → video for any animation
7. `vercel --prod` → `curl -sI <url> | head -1` → 200

## Supabase Auth Pattern
```
lib/supabase/server.ts  — Server Components, reads cookies, NEVER import client-side
lib/supabase/client.ts  — browser only, no service role
lib/supabase/admin.ts   — service role, server-side ONLY, never exported to client
```
Mixing these causes silent 401s in production. See [[01-rules/autonomous-operations]].

## SRI Exception: Google Analytics (gtag.js)
Google's `gtag.js` cannot use SRI (Subresource Integrity). Google dynamically updates the file — any hash breaks it.
```html
<!-- CORRECT — no integrity attribute -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXX"></script>

<!-- WRONG — will break when Google updates the file -->
<script async src="..." integrity="sha384-..."></script>
```
The security hook (PostToolUse:Write) will flag the missing integrity attribute. Dismiss — this is the correct exception.
Applies to: all sites using GA4. See [[brain/Key Decisions]] — SRI Exception entry.
