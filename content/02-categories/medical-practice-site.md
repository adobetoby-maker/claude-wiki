# Category: Medical Practice Website

Used by: orthobiologic-pathways, tobyandertonmd, anderton-and-associates, ime-coach, future medical sites.
Compose with nextjs-supabase-saas or nextjs-cloudflare-deploy for stack details.

## Core Principle: Trust and Compliance First

Medical sites are held to a higher bar than general business sites. Every design and content choice runs through three filters:

1. **Trust** — visitors are often anxious, injured, or researching for a loved one. Tone is calm, authoritative, and human. No marketing hype.
2. **Accuracy** — medical claims must be substantiated. Every benefit statement needs a source or hedge ("may help", "studies suggest").
3. **Compliance** — HIPAA touches anything involving patient data. ADA/WCAG AA is non-negotiable.

## HIPAA Boundaries

The site itself usually does NOT handle PHI (protected health information). PHI lives in the EHR. Common touch points to flag:

- **Contact forms** — name + general inquiry is fine. Symptoms, conditions, diagnoses → NOT through the public form. Direct to phone or a HIPAA-secure portal.
- **Appointment requests** — collect minimal info. Don't ask for diagnosis on the form.
- **Email** — no PHI via standard email. Use a HIPAA-secure messaging tool (e.g. Spruce, Tebra, MyChart).
- **Analytics** — Google Analytics on PHI-adjacent pages requires careful review. Prefer first-party analytics.

When in doubt, treat the data as PHI and route through a BAA-covered service.

## ADA / WCAG AA Requirements

- Color contrast 4.5:1 for body text, 3:1 for large text
- All interactive elements keyboard-navigable
- Form labels associated with inputs (htmlFor / aria-label)
- Alt text on images that convey information
- Skip-to-content link at top of every page
- No motion that violates `prefers-reduced-motion`
- Headings in semantic order (no h1 → h3 skips)

Run axe-core or Lighthouse audit before deploy.

## Standard Page Structure

Most medical sites need this minimum:

```
/                    → Home (hero, services overview, primary CTA)
/about               → Practitioner bio + credentials
/services            → Services index
/services/[slug]     → Individual service page
/conditions          → Conditions index (optional)
/conditions/[slug]   → Condition + treatment options
/locations           → If multiple sites
/insurance           → Accepted plans + verification CTA
/new-patient         → New patient forms / intake info
/contact             → Phone, address, secure-message link
/blog                → Educational content
/privacy             → HIPAA notice of privacy practices
/accessibility       → ADA statement
```

## Schema.org Markup (Required)

```typescript
// app/(site)/layout.tsx — include JSON-LD on every page
const schema = {
  "@context": "https://schema.org",
  "@type": "MedicalBusiness",
  "name": "Dr. Toby Anderton MD",
  "physicianSpecialty": ["..."],
  "address": { ... },
  "telephone": "...",
  "url": "...",
  "openingHours": "...",
  "sameAs": [/* social profiles */]
}
```

Per-service pages get `MedicalProcedure` or `MedicalTherapy`. Per-condition pages get `MedicalCondition`. Use the most specific schema type available.

## Content Voice

- **Read level:** 8th grade. Tools: Hemingway Editor, Yoast readability.
- **Person:** Second person ("you can expect...") for patient-facing. Third person ("Dr. Anderton specializes in...") for bio pages.
- **Avoid:** "Revolutionary", "cutting-edge", "breakthrough" — flags as marketing fluff and erodes trust.
- **Use:** "Evidence-based", "studied", "FDA-approved" — only if true and citable.
- **Hedge medical claims:** "may help", "evidence suggests", "in some patients" — unless you have a clinical citation.

## Practitioner Bio Requirements

Every medical site has at least one practitioner bio. Required fields:

- Full name + credentials (MD, DO, PhD, etc.)
- Medical school + residency + fellowships
- Board certifications with year
- Years in practice
- Office address(es) and phone
- Insurance accepted (link to list)
- High-resolution professional photo
- A short "about" paragraph that includes one human detail (family, hobby, language spoken)

## Required Trust Signals

Place at minimum one of each above the fold on the home page:

- Patient testimonial (with name + city if consented)
- Star rating widget (Google Reviews, Healthgrades, Zocdoc)
- Board certification logos
- Years in practice statement
- "[Practitioner] is accepting new patients" badge if true

## Contact Form Convention

Minimal field set to stay HIPAA-clean:

```typescript
type ContactInquiry = {
  name: string
  email: string
  phone: string
  preferredContact: 'email' | 'phone'
  inquiryType: 'new-patient' | 'existing-patient' | 'billing' | 'other'
  // NO: symptoms, conditions, diagnoses, medications, insurance details
  generalMessage: string  // labeled as "non-medical inquiry only"
}
```

Form copy MUST include: "Please do not include medical information in this form. For health questions, call [number] or use our secure patient portal."

## SEO Patterns Specific to Medical

- **Local SEO is everything.** City + condition + procedure combinations dominate.
  - "Knee pain treatment Twin Falls Idaho"
  - "PRP therapy near Magic Valley"
- **NAP consistency** — Name, Address, Phone must match exactly across Google Business Profile, site, Healthgrades, Yelp.
- **Reviews are ranking factors.** Build a review-request flow into post-visit communication (out of scope for the site, but mention in the project plan).
- **E-E-A-T matters disproportionately.** Author byline with credentials on every blog post. Link to medical sources (NIH, Mayo, peer-reviewed journals).

## Failure Modes

- Collecting symptoms through a public form → HIPAA exposure. Pull the field, add a notice.
- Marketing-flavored medical claims → trust erosion + potential FTC issue. Hedge or remove.
- Missing accessibility audit before launch → lawsuits are common, fix at build time not after.
- Using stock photos of "doctors" instead of the actual practitioner → trust killer.
- Forgetting to set up Google Business Profile → invisible in local search.
- HTTP form submission without HTTPS lock → visible browser warning, instant trust loss.

## What These Projects Tend NOT to Need

- Ecommerce — most medical practices don't sell products through the site
- User accounts (unless porting from MyChart-style portal) — usually handled by the EHR
- Heavy animation — calm > flashy
- Chatbots — not a fit for medical inquiry triage

## Lifecycle Defaults

Most medical sites move to `maintenance` lifecycle within 3 months of launch. They get small content updates (new services, blog posts) but rarely structural changes. Mark accordingly in project CLAUDE.md.
