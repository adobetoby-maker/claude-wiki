# Category: Marketing Site

Used by: tobyandertonmd, willie-elam, salvorias, climb-brasil, climb-spain, climb-utah, climb-kalymnos, future single-purpose marketing sites.
Compose with nextjs-vercel-deploy or nextjs-cloudflare-deploy for stack specifics.

## Definition

A "marketing site" is a Next.js project whose entire purpose is to convert visitors. Defined by what it lacks:

- No customer accounts or login
- No application database (Supabase may exist for forms/leads, but no user tables)
- No `/admin` and `/portal` dual-auth complexity
- No long-running session state

Most are 5–20 pages of content + a small set of forms or CTAs. Lifecycle moves to `maintenance` within 60 days of launch.

## Stack Defaults

- Next.js 16 App Router
- React 19
- Tailwind CSS v4
- TypeScript strict
- Framer Motion for transitions and scroll animations (optional)
- React Three Fiber for 3D heroes (rare, project-by-project)
- Deployment: Vercel by default — Cloudflare only if DNS forces it

## Directory Layout

```
app/
  (site)/          → all public pages live here
    page.tsx       → home
    about/page.tsx
    services/page.tsx
    contact/page.tsx
    blog/[slug]/page.tsx
  api/             → form handlers only (contact, newsletter)
components/
  marketing/       → hero, features, cta, footer
  forms/           → form components with validation
content/
  articles.ts      → blog content (TS array — see ADR 0004)
  services.ts      → service descriptions
  testimonials.ts  → social proof data
lib/
  schema.ts        → JSON-LD helpers
  analytics.ts     → tracking config
```

No `lib/supabase/`. No `lib/adminAuth.ts`. No `proxy.ts` middleware unless you have redirects.

## Required Pages

| Page | Path | Purpose |
|---|---|---|
| Home | `/` | Hero + value prop + primary CTA |
| About | `/about` | Trust, founder/team story |
| Services or Products | `/services` or `/products` | What you sell |
| Contact | `/contact` | Phone, address, form |
| Privacy | `/privacy` | Required for any form collection |
| Terms | `/terms` | Optional but recommended |
| 404 | `app/not-found.tsx` | Custom 404 with home CTA |

## Performance Targets

Marketing sites live or die on Core Web Vitals.

| Metric | Target | How |
|---|---|---|
| LCP | < 2.0s | Hero image: `next/image` with `priority` + `fetchPriority="high"` + `sizes` |
| INP | < 200ms | Avoid hydration of static sections; use `next/dynamic` with `ssr:false` only for true client widgets |
| CLS | < 0.05 | Always set `width`/`height` on images; reserve space for above-the-fold dynamic content |
| TTFB | < 600ms | ISR or static — never SSR a marketing home page |
| Bundle | First-load JS < 100kb | Audit `next build` output, lazy-load anything below the fold |

## Image Handling

- All images go through `next/image`
- Hero image: WebP + AVIF, served via the Image Optimization API (Vercel) or Cloudflare Images
- Lazy-load everything below the fold (`loading="lazy"` — default in next/image)
- Sizes prop is mandatory: `sizes="(max-width: 768px) 100vw, 50vw"` etc.
- Decorative images get `alt=""`. Content images get descriptive alt text.

## SEO Defaults (Required)

Every page needs:

```typescript
// app/(site)/<page>/page.tsx
export const metadata: Metadata = {
  title: 'Page Title — Brand Name',
  description: '150–160 char meta description',
  alternates: { canonical: 'https://example.com/path' },
  openGraph: {
    title: ...,
    description: ...,
    url: ...,
    images: [{ url: '/og.png', width: 1200, height: 630 }],
  },
  twitter: {
    card: 'summary_large_image',
    title: ...,
    description: ...,
    images: ['/og.png'],
  },
}
```

Plus globally:

- `app/sitemap.ts` — generated from all routes + articles
- `app/robots.ts` — env-gated allow/deny
- `app/(site)/layout.tsx` — JSON-LD Organization + LocalBusiness schema
- Open Graph image at `/og.png` (1200×630)

## Conversion Patterns

Above-the-fold elements that every marketing home page needs:

1. **One-line value prop** — what you do, for whom, in 8 words
2. **Sub-headline** — the unique angle in 20 words
3. **Primary CTA button** — single, prominent, action verb
4. **Secondary CTA or social proof** — review badge, customer logos, "as seen in"
5. **Trust signal** — years in business, reviews count, certifications

Below the fold: features grid → testimonials → CTA repeat → FAQ → final CTA → footer.

## Form Handling

Contact and newsletter forms only. Two options:

**Option A — Server Action (preferred):**
```typescript
// app/(site)/contact/page.tsx
'use client'

async function handleSubmit(formData: FormData) {
  'use server'
  // validate + send email or save lead
}
```

**Option B — API Route Handler:**
```typescript
// app/api/contact/route.ts
export async function POST(req: Request) {
  // validate + send + redirect
}
```

Spam protection: Vercel BotID (preferred), Cloudflare Turnstile, or a honeypot field. Never recaptcha v2 (terrible UX).

## Analytics

Default stack:

- Vercel Analytics (built-in, no script tag)
- Vercel Speed Insights (Core Web Vitals in dashboard)
- PostHog or Plausible for product analytics (privacy-friendly, no consent banner needed in most jurisdictions)

Avoid Google Analytics 4 unless the client demands it — needs consent banners in EU, adds bundle weight.

## Animation Patterns

Use Framer Motion sparingly. Common patterns:

- Fade-in on scroll: `motion.div` with `whileInView` + `viewport={{ once: true }}`
- Stagger children: `staggerChildren` in parent `variants`
- Hero parallax: `useScroll` + `useTransform`
- Page transition: wrap layout in `AnimatePresence` (App Router needs `mode="wait"`)

ALWAYS respect `prefers-reduced-motion`:

```typescript
import { useReducedMotion } from 'framer-motion'
const shouldReduceMotion = useReducedMotion()
```

## Animation Verification Rule

For ANY animated, scroll-driven, hover, or parallax UI: use `record.js`, NOT `screenshot.js`. Screenshots freeze mid-animation and produce false-positive quality scores.

```bash
node ~/record.js <port>              # 30s scroll-through → /tmp/preview/review.mp4
node ~/record.js <port> --mobile     # 390×844 iPhone viewport
node ~/record.js <port> --slow       # 60s for detailed motion review
```

## Blog (Optional)

If the project has a blog, use `lib/articles.ts` TypeScript array, not markdown files (see ADR 0004).

```typescript
export const articles = [
  { slug, title, excerpt, body, date, author, tags, readTime },
] as const
```

Render at `app/(site)/blog/[slug]/page.tsx`.

## What Marketing Sites DON'T Do

- No real authentication
- No user-generated content
- No real-time features
- No GraphQL — REST or Server Actions only
- No state management library — RSC + Context where needed
- No client-side data fetching — fetch in RSC, pass as props
- No `useEffect` for data loading — use RSC

## Failure Modes

- LCP > 2.5s because hero image isn't `priority` → biggest preventable SEO loss
- Missing `sizes` prop on next/image → wrong image size served, CLS spikes
- Animation without `prefers-reduced-motion` check → accessibility complaint
- Form submits without honeypot/BotID → spam floods inbox within a week
- Missing `app/sitemap.ts` → Google never finds the deep pages
- Hardcoded canonical URL pointing at preview deployment → indexes the preview, not prod
- Using `next/script` for analytics in `<head>` without `strategy="afterInteractive"` → blocks LCP
- Verifying animations with screenshots instead of video

## Decision Defaults

| User says | Default |
|---|---|
| "add a page" | Create under `app/(site)/<slug>/page.tsx` |
| "add a blog post" | Edit `content/articles.ts`, no new file |
| "improve speed" | Open `next build` output, check LCP image first |
| "add animation" | Framer Motion `whileInView` with `viewport.once` |
| "make it convert better" | Check above-the-fold elements list first |
| "verify the change" | Animations → record.js. Static → screenshot.js |
