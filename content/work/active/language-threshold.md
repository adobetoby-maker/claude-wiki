---
ai-first: true
type: project
date: 2026-05-28
tags: [language-threshold, medicalspanish, constructionspanish, juniorlinguist, language-learning]
status: active
---

# Language Threshold Apps

## For future Claude
Three language-learning apps under the Language Threshold brand. All use Vite SPA + prerender setup. GA tag `G-RP0TZ1MP7E` is on all three (as of 2026-05-28).

---

## medicalspanish.app
- **Path:** `/Users/drive/medicalspanish-app`
- **Vercel URL:** medicalspanish-app.vercel.app (live, returns 200)
- **Custom domain:** medicalspanish.app — HTTP/2 200 ✓ (403 cleared by new clean deploy 2026-05-30)
- **GA:** G-RP0TZ1MP7E ✅ (wired in index.html)
- **Stack:** HTML5 + Three.js standalone (no bundler) — rebuilt 2026-05-30. Old Vite/React app in `src/` preserved but not loaded.
- **Email capture:** `api/email-capture.ts` — CONFIRMED working ✓
- **Monetization:** Outbound affiliate/app links only. PDF Stripe: `https://buy.stripe.com/aFaaEZfqj4hDbah4YebfO07`. Hospital B2B demo: `https://cal.com/medicalspanish/hospital-demo`.
- **Supabase:** email capture (contacts list only)

### HTML5 + Three.js Rebuild (2026-05-30) — DEPLOYED ✓

**Design tokens:** teal `#14B8A6`, navy background, white text. Fonts: Inter (body) + display.

**Three.js pulse network:**
- 100 teal particle nodes in [-50, 50]^3 space, 40 inner core "pulse ring" particles
- Slow scene rotation, mouse parallax, breathing opacity animation
- Canvas opacity: `0.55`
- Loaded via ES importmap: `three@0.165.0` from jsdelivr

**Hero image:** `public/hero-bg.jpg` — fal.ai generated (two X-ray skull profiles facing each other, glowing cyan brains visible, dark navy background). Style: `background: url('/hero-bg.jpg') center 30%/cover no-repeat`.

**Sections:** Nav → Hero (canvas) → Problem + 3 emergency translation tools → 9 sample phrases → 9 module cards (linked to specific article pages) → Trifold email form → PDF $4.99 Stripe banner → CE/CME section (Relias, Medscape, Coursera for Healthcare) → Hospital B2B (ACA Section 1557 framing) → 40 articles grid (10 live + 30 coming-soon stubs) → Final CTA → Footer

**vercel.json** (deployed 2026-05-30):
```json
{
  "buildCommand": null,
  "outputDirectory": ".",
  "rewrites": [
    { "source": "/api/(.*)", "destination": "/api/$1" }
  ]
}
```

**package.json build script:** `"build": "echo 'static html — no build needed'"` — prevents Vercel from running the old Vite bundler.

**Articles:** 10 live prerendered pages (copied from `dist/articles/`) + 30 coming-soon stubs in the grid (cardiology, oncology, ICU, surgical, radiology, PT, OT, neurology, ortho, GI, pulm, nephrology, endo, derm, rheum, urology, RT, nutrition, social work, case mgmt, ophtho, ID, wound care, home health, dental, ultrasound, lab, allergy, interpreting, documentation).

**medicalspanish.app 403 — RESOLVED:** Was Vercel Attack Challenge Mode. New clean deploy (2026-05-30) cleared it automatically. No dashboard action required after all.

### Pending
- [ ] **Link audit** — run Chrome DevTools MCP audit on live medicalspanish.app (same method as construction audit). Swarm agent assigned but confirmation pending.
- [ ] **Fix .html extension links** — nav and article links likely use `.html` extension; prerendered pages are directories (need `/for-doctors/` not `/for-doctors.html`)
- [ ] **30 stub articles → full content** — cardiology, oncology, ICU, surgical, radiology, PT, OT, neurology, ortho, GI, pulm, nephrology, endo, derm, rheum, urology, RT, nutrition, social work, case mgmt, ophtho, ID, wound care, home health, dental, ultrasound, lab, allergy, interpreting, documentation

## constructionspanish.app
- **Path:** `/Users/drive/constructionspanish-app`
- **Vercel URL:** constructionspanish-app.vercel.app (live, deployed)
- **Custom domain:** constructionspanish.app (DNS pending — needs A record → 76.76.21.21)
- **GA:** G-RP0TZ1MP7E ✅ (wired)
- **Stack:** HTML5 + Three.js (standalone, no bundler) — `index.html` is the full app entry point as of 2026-05-29 rebuild. React components still exist in `src/` but are NOT loaded by the new `index.html`.
- **Design tokens:** `text-orange` (#FF6B2B), `text-cream` (#F5F0E8), `bg-concrete` (#2C2C2C), `bg-concrete-dark` (#1E1E1E). Fonts: Barlow Condensed (display, uppercase bold) + IBM Plex Sans.

### Full Rebuild (2026-05-29)

**New components built:**
- `LTDemoSection.tsx` — 4-tab browser-chrome mockup of Language Threshold app (Vocabulary Drill, Conjugation for `levantar`, Phrases, Progress)
- `ModuleCTAs.tsx` — 6 LT construction modules with lesson counts, sample phrases, CTA → `https://app.languagethreshold.com/construction/<module-id>`
- `TrifoldCTA.tsx` (rewritten) — email capture lead magnet; POST `/api/email-capture` with `source: 'trifold-construction'`; success shows download link to trifold PDF
- `PocketGuideBanner.tsx` — PDF companion $4.99 Stripe link (`buy.stripe.com/dRm14pa5Z15r0vDduKbfO06`)
- `OshaAffiliateCTA.tsx` — 3 OSHA platform cards (OSHA Ed Center, NCCER, 360training); affiliate hrefs pending applications
- `JobsiteToolsCTA.tsx` — 3 translation tools (WT2 Edge earbuds, Pixel Buds, Google Translate); Amazon affiliate tag `climbing00bb-20` (needs updating)
- `Dictionary.tsx` — full-page at `/dictionary` route; search + PoS filter; WordCard with conjugation table + usage examples

**Data files:**
- `src/data/constructionWords.ts` — 2,611 lines, full construction vocabulary with verb conjugation tables, from lt-dictionary
- `src/types/dictionary.ts` — Word, VerbProfile, MorphForm interfaces

**App.tsx component order (homepage):**
`Hero → WhoItsFor → TheProblem → HowItWorks → LTDemoSection → ModuleCTAs → SamplePhrases → SampleLesson → InlineDictionary → TrifoldCTA → PocketGuideBanner → AmazonBooks → EmailCaptureSection → B2BCTA → FinalCTA`

**API:** `api/email-capture.ts` (Vercel Function) — Resend email + Worker Bee contacts sync (`siteId: 'constructionspanish'`)

**KAIZEN.md status:**
- Closed: full rebuild with all new components
- Open: SEO articles sprint (10→40), trifold PDF hosting, OSHA affiliate applications, Amazon tag update

### SEO Articles Sprint (2026-05-29) — COMPLETED ✓

- **Result:** 40 articles total (10 original + 30 new) — matches medicalspanish.app target
- **Build:** Passes cleanly, 41 routes prerendered (index + 40 articles)
- **Deploy:** `vercel --prod` LIVE — `curl -sI https://constructionspanish.app` returns `HTTP/2 200` ✓ (as of 2026-05-29)
- **30 new articles:** landscaping, HVAC, masonry, painting, welding, heavy equipment, demolition, scaffolding, project manager, construction math, tile/flooring, windows/doors, steel erection, excavation, insulation, waterproofing, emergency, crew leaders, punch list, commercial, solar installation, concrete pumping, daily briefing, fire protection, building inspection, residential, site layout, concrete flatwork, carpentry, Spanish numbers

**Article format standard:**
- 2–3 term tables (English/Spanish, 10–12 rows each) covering subtopic areas
- 10–12 foreman phrases in Spanish with English translation
- LT CTA section linking to `https://app.languagethreshold.com/pricing`
- Back link to articles index

**Key technical pattern — split-brain prerender:**
- `App.tsx` controls SPA routing (all pages work as SPA immediately)
- `scripts/seo-routes.mjs` controls SEO prerendering (opt-in per route, single source of truth for per-route title/description/og/canonical/breadcrumb JSON-LD)
- Without a `seo-routes.mjs` entry, a page is SPA-only — Googlebot sees only the generic `<head>` from `dist/index.html`
- `scripts/prerender.mjs` — custom static head injection (NOT vite-plugin-prerender or react-snap — both incompatible with React 19 + Vite 8)

**Import typo (cosmetic):** Agent named concrete-flatwork import `ConcreteFlatwarkSpanish` (typo in variable name only — file path is correct, no functional impact)

### Email Capture `/api/email-capture` — Investigation (2026-05-29)

**Root cause (initial):** `rewrites` in `vercel.json` were evaluated BEFORE Vercel Function routing. Fixed by switching to `routes` array.

**Root cause (actual, confirmed):** Cloudflare Pages project for `constructionspanish.app` still has the custom domain configured. Cloudflare Pages Workers attach at the **zone edge layer**, intercepting ALL traffic to the domain before DNS can forward to Vercel — even when A/AAAA records point to Vercel's IPs.

**Evidence confirming this:**
- Custom domain serves JS hash `index-DhHwwseS.js` (old Pages build)
- Latest Vercel deployment has `index-Bqz7yxSw.js` — different hash proves Pages is serving stale content
- `POST constructionspanish-app.vercel.app/api/email-capture` → `{"ok":true}` ✓ (bypasses Cloudflare — function works)
- `POST constructionspanish.app/api/email-capture` → 405 empty body (Pages Worker intercepts, has no `api/` function)
- `cf-cache-status: DYNAMIC` in response headers does NOT confirm the request reached Vercel — Pages Workers generate this header for their own responses

**Current deployed `vercel.json`** (both sites, as of 2026-05-29):
```json
{
  "routes": [
    { "handle": "filesystem" },
    { "src": "/(.*)", "dest": "/index.html" }
  ]
}
```
The `{ "handle": "filesystem" }` sentinel tells Vercel's edge to check static files and functions before applying catch-all. Explicit `/api/(.*)` route was removed — filesystem handler auto-discovers `api/email-capture.ts`. This is correct and will work once Cloudflare Pages no longer intercepts.

**Fix required (manual — Cloudflare dashboard):**
1. dash.cloudflare.com → Workers & Pages → Pages → `constructionspanish-app` → Custom domains → Remove `constructionspanish.app`
2. Same for `medicalspanish-app` → Remove `medicalspanish.app`
3. After removal: `curl -si -X POST https://constructionspanish.app/api/email-capture -H "Content-Type: application/json" -d '{"email":"smoke@test.com","source":"trifold-construction"}'` → must return `{"ok":true}`

**Cloudflare builds MCP:** OAuth flow started (`cloudflare-builds` plugin) — authorization URL was provided to user but completion not confirmed. Cloudflare API MCP token (separate) was expired throughout investigation.

### HTML5 + Three.js Rebuild (2026-05-29) — DEPLOYED ✓

**Approach:** Standalone HTML5 page replaces Vite/React SPA entry point. Zero build tooling — Three.js loaded via ES `importmap` from CDN.

**Three.js scaffolding network:**
```javascript
import * as THREE from 'three'; // loaded via importmap from cdn.jsdelivr.net/npm/three@0.165.0
// 120 orange (#FF6B2B) particle nodes scattered in [-55, 55]^3 space
// LineSegments connect nodes within 20 units of each other
// GridHelper: blueprint floor at y=-30
// Mouse parallax: camera.position.x += (mx * 6 - camera.position.x) * .04
// Breathing: ptMat.opacity = .55 + Math.sin(t * .8) * .2
```

**Sections (all in one HTML file):** Nav → Hero (canvas behind content) → Problem + 3 emergency translation tools → 9 sample phrases grid → 9 training modules + "more trades" CTA → Trifold email form → PDF $4.99 Stripe banner → 4 Amazon books → 3 OSHA cert platforms → B2B panel → 40 articles grid → Final CTA → Footer

**Email form (vanilla JS):**
```javascript
async function submitTrifold() {
  var res = await fetch('/api/email-capture', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email: email, name: '', source: 'trifold-construction' })
  });
}
```

**vercel.json** (deployed 2026-05-30):
```json
{
  "buildCommand": null,
  "outputDirectory": ".",
  "rewrites": [
    { "source": "/api/(.*)", "destination": "/api/$1" }
  ]
}
```

**package.json build script:** `"build": "echo 'static html — no build needed'"` — prevents Vercel from running the old Vite bundler.

**Hero image:** `public/hero-bg.jpg` — fal.ai generated (construction crew in orange hard hats on high-rise rooftop at golden hour, two tower cranes, city skyline). Style: `background:url('/hero-bg.jpg') center 40%/cover no-repeat`.

**Module cards fix (2026-05-30):** Converted all 9 module `<div class="mc">` elements to `<a class="mc">` with real article hrefs:
- `/articles/concrete-framing-spanish`, `/articles/100-phrases-foremen`, `/articles/osha-spanish`
- `/articles/plumbing-spanish`, `/articles/electrical-spanish`, `/articles/drywall-carpentry-spanish`
- `/articles/landscaping-spanish`, `/articles/heavy-equipment-spanish`, `/articles/welding-spanish`

**Link audit (2026-05-30):** Chrome DevTools MCP JavaScript audit of live constructionspanish.app found 91 links + 1 button. Two systemic issues found:
1. **`.html` extension on internal nav/footer links** — nav uses `/for-foremen.html`, `/for-contractors.html`, `/safety.html`, `/dictionary.html` but prerendered pages are directories
2. **`.html` extension on 40 article grid links** — e.g., `/articles/electrical-spanish.html` should be `/articles/electrical-spanish/`
3. **`dictionary/` page missing** — was a client-side-only React route, never prerendered; `dictionary/index.html` does not exist

### Pending
- [x] ~~Verify Vercel deployment~~ — CONFIRMED `HTTP/2 200` ✓ (2026-05-29)
- [x] ~~Cloudflare Pages domain removal~~ — DONE (2026-05-29). User removed `constructionspanish.app` from Pages project + `medicalspanish.app` from its Pages project via Cloudflare dashboard. Email capture on both custom domains now returns HTTP/2 200 `{"ok":true}` ✓
- [x] ~~Smoke test email capture~~ — PASSED ✓ (2026-05-29). Both custom domains confirmed working post-Cloudflare fix.
- **Trifold** — NOT a PDF file. Print-in-browser React app at `languagethreshold.com/print/trifold/construction`. 9 modules: auto-mechanic, drywall, electrician, foreman, framer, landscaper, plumber, safety, truck-driver. Routes registered in `App.tsx`. `TRIFOLD_PDF_URL` in `TrifoldCTA.tsx` points to this URL. ✓ CLOSED
- [ ] Verify Stripe PDF Companion link at `buy.stripe.com/dRm14pa5Z15r0vDduKbfO06`
- [ ] Apply to OSHA Education Center, NCCER, 360training affiliate programs; update `OshaAffiliateCTA.tsx`
- [ ] **Amazon affiliate tag DEFERRED permanently** — `climbing00bb-20` in `AmazonBooks.tsx:28` + `JobsiteToolsCTA.tsx:5` needs a construction/language Associates tag, not a climbing tag. Requires applying for a new Associates program. No immediate action needed.
- [x] ~~**Fix .html extension links in index.html**~~ — DONE (2026-05-30). Node.js script replaced all 5 nav/footer links + 40 article grid links. 0 remaining `.html` internal links.
- [x] ~~**Create `dictionary/index.html`**~~ — DONE (2026-05-30). Standalone HTML5 searchable dictionary at `/Users/drive/constructionspanish-app/dictionary/index.html`. 97 words embedded as JS literal. Search by Spanish/English/pronunciation. Filter: All/Verbs/Nouns/Phrases. XSS-safe via `esc()` helper. SEO: JSON-LD `DefinedTermSet`. CTA → `https://app.languagethreshold.com/pricing`.
- [ ] **Deploy** — `cd /Users/drive/constructionspanish-app && vercel --prod`, then verify nav + article links return 200.

## juniorlinguist.com
- **Path:** `/Users/drive/juniorlinguist-app` (check actual path)
- **Status:** Auth complete (kid login, email, referral)
- **Stack:** Next.js + Supabase

## DNS Cutover (BLOCKING — user must do in Cloudflare dashboard)
Both medicalspanish.app and constructionspanish.app use Cloudflare nameservers.
Action needed:
1. Cloudflare dashboard → medicalspanish.app → DNS
2. A record: `@` → `76.76.21.21` → Proxy: DNS-only (grey cloud)
3. Same for constructionspanish.app
4. Verify: `curl -s https://medicalspanish.app | grep G-RP0TZ1MP7E`

## GA Tag Format (verbatim — do not change)
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

## language-lens-elite
- **Path:** `/Users/drive/language-lens-elite`
- **URL:** app.languagethreshold.com (live)
- **Stack:** Vite + TanStack Start → Vercel (NOT CF Workers, NOT Next.js)
- **Supabase:** `pollhlkgltdkdskdzsgd`
- Known gap: DATABASE_URL/DIRECT_URL have `[YOUR-PASSWORD]` placeholder → Prisma not connected
