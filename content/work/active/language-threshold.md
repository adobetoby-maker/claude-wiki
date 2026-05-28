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
- **Vercel URL:** medicalspanish-app.vercel.app (live, deployed)
- **Custom domain:** medicalspanish.app (DNS pending — needs A record → 76.76.21.21)
- **GA:** G-RP0TZ1MP7E ✅ (wired in index.html + prerendered routes)
- **Stack:** Vite + React + prerender.mjs (17 static routes)
- **Supabase:** stripe webhooks, email capture

## constructionspanish.app
- **Path:** `/Users/drive/constructionspanish-app`
- **Vercel URL:** constructionspanish-app.vercel.app (live, deployed)
- **Custom domain:** constructionspanish.app (DNS pending — needs A record → 76.76.21.21)
- **GA:** G-RP0TZ1MP7E ✅ (wired)
- **Stack:** Vite + React + prerender.mjs

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
