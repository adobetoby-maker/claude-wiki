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

## Vercel SPA + Functions: Use `routes` Not `rewrites`

**Problem:** In Vite SPA (non-Next.js) projects on Vercel, the catch-all `rewrites` rewrite is evaluated BEFORE Vercel Functions. A `{ "source": "/(.*)", "destination": "/index.html" }` rewrite intercepts `/api/*` and returns SPA HTML instead of routing to the serverless function.

**Symptom:** `POST /api/endpoint` returns HTTP 200 with HTML body; Vercel runtime logs show zero function invocations.

**Fix:** Use the deprecated `routes` array (first-match-wins) instead of `rewrites`:
```json
{
  "routes": [
    { "src": "/api/(.*)", "dest": "/api/$1" },
    { "src": "/(.*)", "dest": "/index.html" }
  ]
}
```

**Applies to:** Any Vite/React SPA on Vercel with `api/` serverless functions — `constructionspanish-app`, `medicalspanish-app`.
**Does NOT apply to:** Next.js projects (file-system routing handles this automatically).

## Cloudflare Orange-Cloud + Vercel Functions: Bypass `/api/*`

**Problem:** When a domain uses Cloudflare's orange-cloud proxy and routes to Vercel, Cloudflare may serve a cached HTML response for `/api/*` paths even after Vercel-side routing is fixed. The function works correctly at the `.vercel.app` deployment URL but returns HTML at the custom domain.

**Diagnosis:**
1. Check DNS: `dig constructionspanish.app` — Cloudflare IPs (`172.67.x.x` / `104.21.x.x`) confirm orange-cloud proxy active
2. Test deployment URL directly: `curl -X GET https://<project>-<hash>.vercel.app/api/endpoint` — if this returns JSON, Vercel is fine
3. Test custom domain: `curl -sI https://customdomain.com/api/endpoint | grep content-type` — if `text/html`, Cloudflare is serving cached HTML
4. **Compare JS asset hashes** — `curl -s https://customdomain.com | grep -oE '[a-zA-Z0-9-]+\.js'` vs latest Vercel deployment. Different hash = Cloudflare serving stale build, not just cache.

**Fix options (pick one):**
1. **Cache Rule** (preferred for orange-cloud cache) — Cloudflare dashboard → Rules → Cache Rules → "URI Path starts with /api" → Cache Status: Bypass
2. **Purge cache** — Cloudflare dashboard → Caching → Purge Everything (temporary fix, recurs on next cache fill)
3. **Grey-cloud the domain** — Change DNS record proxy from orange to grey for the domain entirely (loses Cloudflare benefits)

**Applies to:** All Cloudflare-proxied custom domains pointing to Vercel with serverless functions. Confirmed on `constructionspanish.app` and `medicalspanish.app` (as of 2026-05-29).

---

## Cloudflare Pages Zone-Edge Interception (Severe — Not Cache)

**Problem:** If a domain was previously pointed at a Cloudflare Pages project AND that Pages project still has the domain listed as a custom domain, the Pages Worker intercepts ALL traffic at the Cloudflare edge layer — before DNS resolution reaches Vercel. This happens even when A/AAAA records correctly point to Vercel's IPs.

**Key distinction from cache:** This is not caching — this is Cloudflare Pages actively serving the old Pages build. `cf-cache-status: DYNAMIC` does NOT confirm the request reached Vercel; Pages Workers generate this header for their own responses.

**Diagnosis:**
1. Compare JS asset hashes: `curl -s https://customdomain.com | grep -oE 'index-[A-Za-z0-9]+\.js'` — hash differs from latest Vercel deployment → Pages is actively serving a stale build
2. POST to `.vercel.app` alias works, POST to custom domain returns 405 with empty body → the Pages project has no `api/` function, confirming Pages interception
3. Vercel runtime logs show zero invocations from the custom domain

**Fix:** Remove the custom domain from the Cloudflare Pages project:
dash.cloudflare.com → Workers & Pages → Pages → [project-name] → Custom domains → Remove

**Confirmed on:** `constructionspanish.app` and `medicalspanish.app` (as of 2026-05-29). Both had active Cloudflare Pages projects with the custom domains still configured despite migration to Vercel.

## GA4 Deployment: index.html vs Next.js Layout

Two patterns depending on stack. GA4 measurement ID for all Drive sites: `G-RP0TZ1MP7E`.

**Vite SPA — hardcode in `index.html` before `</head>`:**
```html
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-RP0TZ1MP7E"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-RP0TZ1MP7E');
</script>
```
Sites using this: `mtn-edge-plumb-hub`, `language-threshold`, `juniorlinguist`, `lt-dictionary`.

**Next.js App Router — add `Script` component to root `layout.tsx`:**
```tsx
import Script from 'next/script'
// inside <body>:
<Script src="https://www.googletagmanager.com/gtag/js?id=G-RP0TZ1MP7E" strategy="afterInteractive" />
<Script id="gtag-init" strategy="afterInteractive">{`
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-RP0TZ1MP7E');
`}</Script>
```
Sites using this: all Next.js sites including `manage-worker-bee`, `climb-idaho`, `anderton-contracting`, `mountain-edge-*`, `magic-valley-mechanical`.

**Note:** No `integrity` attribute on gtag.js — Google dynamically updates the file. See SRI Exception entry below.

## Vercel REST API for Env Vars (Batch / Programmatic)

When `vercel env add` in a loop operates on the wrong project context (cd + CLI doesn't reliably switch context), use the REST API directly:

```bash
TOKEN=$(cat "/Users/drive/Library/Application Support/com.vercel.cli/auth.json" | python3 -c "import json,sys; d=json.load(sys.stdin); print(list(d.get('tokens',{}).values())[0] if isinstance(d.get('tokens'),dict) else d.get('token',''))")
TEAM="team_WPMJl6w7aYPU9xP3sr8Xx3uN"

curl -s -X POST "https://api.vercel.com/v10/projects/{PROJECT_ID}/env?teamId=$TEAM&upsert=true" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"key":"VAR_NAME","value":"var_value","type":"encrypted","target":["production","preview","development"]}'
```

**Response check:** Success response contains `"created"` key, NOT `"key"` key. Parse accordingly.
**Applies to:** Any situation where multiple projects need the same env var set in one session.

## Three.js via ES Module Importmap (No Bundler)

Load Three.js in a standalone HTML5 page without any build tooling:

```html
<script type="importmap">
  {"imports":{"three":"https://cdn.jsdelivr.net/npm/three@0.165.0/build/three.module.js"}}
</script>
<script type="module">
  import * as THREE from 'three';
  // full Three.js API available — renderer, scene, camera, materials, geometries, etc.
</script>
```

**When to use:** Simple affiliate/SEO/marketing sites that don't need React, auth, or a build step. The importmap lets you `import * as THREE from 'three'` in a module script without npm or a bundler.

**Version pin:** Always specify a version (`three@0.165.0`) — `three@latest` can break when Three.js makes breaking API changes.

**Limitations:** No TypeScript, no tree-shaking, no npm addons unless they also ship as ES modules with a CDN URL. OrbitControls and other Three.js addons: `https://cdn.jsdelivr.net/npm/three@0.165.0/examples/jsm/controls/OrbitControls.js`

**Confirmed on:** `constructionspanish-app/index.html` — 120-node scaffolding particle network (2026-05-29).

## Vercel Static HTML Deployment (No Build Step)

When a site is pure static HTML (not an SPA — each URL maps to an actual file on disk), use this vercel.json pattern to:
1. Skip the bundler entirely (`buildCommand: null`)
2. Serve the repo root as the output directory (`outputDirectory: "."`)
3. Route API functions without a SPA catch-all (`rewrites` with explicit `/api/(.*)` only)

```json
{
  "buildCommand": null,
  "outputDirectory": ".",
  "rewrites": [
    { "source": "/api/(.*)", "destination": "/api/$1" }
  ]
}
```

Also update `package.json` to prevent Vercel's default npm build from running the old bundler:
```json
{ "scripts": { "build": "echo 'static html — no build needed'" } }
```

**Key difference from the SPA pattern (ADR-0014):** SPA needs `{ "handle": "filesystem" }` + SPA catch-all because URLs like `/articles/slug` don't map to real files. Static HTML sites have real files at those paths — no catch-all needed. Only the `/api/(.*)` rewrite is necessary.

**`api/` TypeScript functions:** Still compile automatically regardless of `buildCommand: null`. The `buildCommand` setting only controls the frontend build step.

**Confirmed on:** `constructionspanish-app/vercel.json` and `medicalspanish-app/vercel.json` (as of 2026-05-30).

**Gotcha:** Even with `buildCommand: null` in vercel.json, Vercel will still run `npm run build` if package.json has a build script. The no-op script in package.json is required.

---

## Supabase REST API Fallback (When MCP Token Expires)

When the `plugin:supabase:supabase` MCP token expires mid-task and `supabase db query` fails (requires local Postgres), use the REST API directly with the service role key:

```bash
# 1. Get project ref (20-char alphanum) from Supabase URL in .env.local
REF=$(grep NEXT_PUBLIC_SUPABASE_URL .env.local | grep -oE '[a-z]{20}')
# 2. Get service role key
KEY=$(grep SUPABASE_SERVICE_ROLE_KEY .env.local | cut -d= -f2)

# PATCH a row
curl -s -X PATCH "https://$REF.supabase.co/rest/v1/sites?id=eq.<uuid>" \
  -H "apikey: $KEY" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -H "Prefer: return=minimal" \
  -d '{"notes": "...", "status": "active"}'

# SELECT rows
curl -s "https://$REF.supabase.co/rest/v1/sites?select=id,name,url,status" \
  -H "apikey: $KEY" \
  -H "Authorization: Bearer $KEY"
```

**Gotcha:** If PATCH returns HTTP 400, the payload likely has a field with an invalid enum value. Common issue: `stack: "html5"` — only `nextjs`, `wordpress`, etc. are valid. Drop the offending field and retry with only `notes` + `status`.

**Confirmed on:** manage-worker-bee Supabase project `qnrkifdbkcbacgznoabs` (2026-05-30).

---

## XSS-Safe innerHTML in Static HTML Pages

When using `element.innerHTML = template literal` in vanilla JS, always escape all user-controlled or external data through an `esc()` helper:

```javascript
function esc(s) {
  return String(s)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');
}

// Usage inside innerHTML:
grid.innerHTML = words.map(w => `
  <div class="card">
    <div class="es">${esc(w.spanish)}</div>
    <div class="en">${esc(w.english)}</div>
    <div class="pron">${esc(w.pronunciation)}</div>
  </div>
`).join('');
```

**When to use:** Any `.innerHTML =` assignment where the values come from data arrays (even static ones — data can later be fetched or user-supplied). The security hook on the Write tool will flag raw `${variable}` in innerHTML template literals — add `esc()` immediately.

**Does NOT apply to:** Hardcoded HTML strings with no variable interpolation.

**Confirmed on:** `constructionspanish-app/dictionary/index.html` (2026-05-30).

---

## Parallel Background Agent Swarm

Launch multiple independent agents in a single message to fan out across sites:

```javascript
// All Agent() calls in one message → run concurrently
Agent({ description: "audit site A", run_in_background: true, prompt: "..." })
Agent({ description: "audit site B", run_in_background: true, prompt: "..." })
Agent({ description: "audit site C", run_in_background: true, prompt: "..." })
// ...up to 8-10 at once
```

**When to use:** Same repeatable task (link audit, affiliate placement, lint fix) across multiple independent sites. Each agent: reads site code, makes fixes, builds, deploys, verifies with `curl -sI`.

**Limitations:** Background agents cannot communicate results back to TAC in this turn. Use synchronous `Bash(timeout=600000)` + `jr "task"` when TAC needs the output. Background swarm is fire-and-forget confirmation.

**Confirmed on:** 8-site link cleanup swarm + 8-site affiliate placement swarm (2026-05-30).

---

## Amazon Affiliate Links: Standard Pattern

All Drive sites use tag `climbing00bb-20` until a dedicated tag is set up per niche:

```html
<a href="https://www.amazon.com/s?k=PRODUCT+KEYWORDS&tag=climbing00bb-20"
   target="_blank" rel="noopener noreferrer sponsored">
  Product Name
</a>
<!-- FTC disclosure required on page: "As an Amazon Associate I earn from qualifying purchases." -->
```

**File locations by site:**
- constructionspanish-app: `AmazonBooks.tsx:28` + `JobsiteToolsCTA.tsx:5`
- climb-brasil: `src/lib/gear.ts` (corrected from `CLIMBBRASIL-20` → `climbing00bb-20` on 2026-05-30)

**Note:** `climbing00bb-20` is the placeholder tag. Each niche site should eventually get its own Associates tag. See [[brain/Key Decisions]] — ADR on affiliate standardization.

---

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
