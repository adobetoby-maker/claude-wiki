# Category: Website Build Protocol

Use this protocol for every new site build, redesign, or major iteration.
Compose with a stack category (nextjs-vercel-deploy, nextjs-cloudflare-deploy, etc.) for platform specifics.

---

## Iron Law — Research Before Code

Observable check that fires before Phase 1:
```bash
ls scores.md 2>/dev/null || echo "MISSING"
git log --oneline -1 2>/dev/null || echo "NO COMMITS"
```

`scores.md` is MISSING → Phase 0 runs NOW. No code written until it exists.
NO COMMITS → Phase 0 runs NOW. Regardless of how the request was framed ("continue", "add feature", "finish").

**Platform Decision Gate (runs before any deploy command):**
```bash
# Is this a worker-bee.app project with D1/R2/KV/cron bindings?
cat wrangler.jsonc 2>/dev/null | grep -E "d1_databases|r2_buckets|kv_namespaces"
```
No output (no bindings) → deploy to Vercel (`vercel --prod`).
Has bindings → deploy to Cloudflare Workers. Confirm `CLOUDFLARE_API_TOKEN` in env first:
```bash
env | grep -i CLOUDFLARE_API_TOKEN
```
See ADR-0014 for the 5-option deploy exhaustion sequence.

---

## Phase 0 — Research (Before Any Code)

**Entry criteria (observable):** `ls scores.md` → MISSING or `git log` → NO COMMITS.
**Time budget:** 20–30 minutes.

Steps:
1. Identify the site category (local business, SaaS, institutional, affiliate, portfolio, etc.)
2. Find 3 best-in-class references via WebSearch or `competitor-profiling` skill
3. Scrape / screenshot each: `node ~/screenshot.js` or `firecrawl:firecrawl-scrape`
4. Score on project-relevant dimensions (see `research-first.md`)
5. Build the gap table: reference scores vs our target
6. State the "build better than X" mandate explicitly

**Deliverables:**
- `scores.md` — reference scores + gap analysis
- `iter-00/` — reference screenshots
- Summary paragraph in project CLAUDE.md under `## Competitive Context`

**Exit criteria:** `ls scores.md` exits 0. Target scores defined. Code not touched yet.

---

## Phase 1 — Architecture & Content Plan

**Entry criteria:** `ls scores.md` exits 0.

Steps:
1. Invoke `brainstorming` skill — generate 3 structural directions
2. Choose direction, document rationale
3. Define route structure (all pages)
4. Define content inventory: what copy, images, data for each page
5. Define component inventory: what UI components needed
6. Map to the relevant category file (marketing-site, nextjs-supabase-saas, etc.)
7. Document deviations from category defaults

**Deliverables:**
- Route map
- Component list
- Content gaps (what client needs to provide)
- Initial CLAUDE.md scaffold with frontmatter

---

## Phase 2 — Design Direction

**Entry criteria:** Architecture locked.

Steps:
1. Invoke `ui-ux-pro-max` skill for design direction
2. Build above-the-fold section first (hero)
3. Run visual verification immediately:
   ```bash
   node ~/record.js <port>          # scroll video — REQUIRED for any animation
   node ~/screenshot.js <port>      # static capture
   ```
4. Read the PNG files with Read tool. Describe what is visible.
5. Score hero against Phase 0 gap table. If hero score doesn't close the gap → iterate before building rest of site.

**Exit criteria:** Hero passes visual gate (PNG + video captured and READ). Score ≥ target on hero impact.

---

## Phase 3 — Full Build

**Entry criteria:** Hero and design system locked.

Steps:
1. Build page by page, section by section
2. After each page: observable check — `git diff HEAD --name-only | grep -E '\.(tsx|css)'` → non-empty → run Gate 2
3. Mobile check after every UI iteration: `node ~/record.js <port> --mobile`
4. No advancing to next page until current page passes visual gate

**Skills:** `nextjs-best-practices`, `nextjs-app-router-patterns`, `react-best-practices`, `shadcn`, `tailwind-patterns`

**Exit criteria:** All pages built. `npx tsc --noEmit` → 0 errors. `npm run build` → succeeds. All visual gates passed.

---

## Phase 4 — SEO Implementation

**Entry criteria:** Build complete, pages rendering correctly.

Steps:
1. Invoke `seo-technical` skill — audit current state
2. Implement metadata on every page (title, description, canonical, OG, Twitter)
3. Add JSON-LD: Organization/LocalBusiness on home, Article on blog posts
4. Generate: `app/sitemap.ts`, `app/robots.ts`
5. Invoke `seo-aeo-schema-generator` for rich snippet opportunities
6. Run Gate 3 verification:
   ```bash
   curl -I <url>/sitemap.xml | head -1   # must be 200
   curl -I <url>/robots.txt | head -1    # must be 200
   ```

**Exit criteria:** Gate 3 passes completely. `curl -I` on sitemap + robots returns 200.

---

## Phase 5 — Performance & Accessibility

**Entry criteria:** SEO implementation complete.

Steps:
1. Check hero image: `grep -r "fetchPriority\|priority" app/ | grep -i hero`
2. Check `prefers-reduced-motion` in any Framer Motion or CSS animation
3. Run `accessibility-compliance-accessibility-audit` skill
4. Fix any WCAG 2.1 AA failures

**Exit criteria:** Gate 4 passes. No WCAG AA failures.

---

## Phase 6 — Deploy & Verify

**Entry criteria:** All previous phases complete.

**Platform Decision (mandatory before any deploy):**
```bash
# Marketing/affiliate/portfolio → Vercel
vercel --prod

# worker-bee.app + D1/R2/KV → Cloudflare Workers (after confirming CLOUDFLARE_API_TOKEN)
npx wrangler deploy
```

After any deploy exits 0:
```bash
curl -sI <live-url> | head -1    # must be HTTP 200
```

If deploy fails → Iron Law 2 (ADR-0014): run 5-option check, report what worked not what failed.

**Exit criteria:** `curl -I <live-url>` → 200. Blueprint pushed to manage.worker-bee.app.

---

## Phase 7 — Maintenance Handoff

**Entry criteria:** Site live and passing all gates.

Steps:
1. Run Gate 7 (from `quality-gate.md`)
2. Update project CLAUDE.md: `lifecycle: active` + `last_verified: <today>`
3. Push memory entry with key decisions

**Exit criteria:** Gate 7 complete. CLAUDE.md written to rubric standard. Memory pushed.

---

## Copywriting — No Placeholder Copy

| Page type | Skill |
|---|---|
| Home hero + value prop | `copywriting` skill |
| Service/product descriptions | `seo-content-writer` skill |
| Blog posts | `seo-aeo-blog-writer` skill |
| Meta descriptions | `seo-aeo-meta-description-generator` skill |

---

## Image Strategy

1. Hero: ComfyUI via `comfy:gen` or `/api/image-gen` — photorealistic or brand-appropriate
2. OG image: 1200×630 branded image via ComfyUI
3. All images: WebP, descriptive alt text, `sizes` prop in next/image
4. ADR-0013: no duplicate image per slot per site

```bash
# Verify uniqueness before declaring done
grep -oE 'photo-[A-Za-z0-9]+|/images/routes/[^'"'"']+' src/lib/routes.ts | sort | uniq -d
# Expected: zero output
```

---

## Complete Deliverable Checklist

```
□ scores.md exists — research done
□ npx tsc --noEmit → 0 errors
□ npm run build → succeeds
□ Visual gate: PNG + video for each page (desktop + mobile), all Read with Read tool
□ SEO: metadata, schema, sitemap.xml (200), robots.txt (200)
□ curl -I <live-url> → 200 and HTTPS
□ No placeholder copy
□ Images: hero, OG image, no broken images, no duplicates
□ CLAUDE.md: frontmatter complete, lifecycle: active, last_verified current
□ Blueprint pushed to manage.worker-bee.app
□ Memory entry pushed
□ 30/60/90-day maintenance plan
```
