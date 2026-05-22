---
title: Drive's Claude Wiki
---

# Drive's Claude Wiki

Personal operating system for Claude Code — rules, protocols, deployment patterns, and project context for the Worker Bee ecosystem.

## Sections

### [[01-rules/autonomous-operations|Rules]]
Iron Laws and enforcement protocols that govern every Claude session.

| Rule | What it enforces |
|---|---|
| [[01-rules/autonomous-operations\|Autonomous Operations]] | When to act vs. pause; 5-method rule; model routing |
| [[01-rules/quality-gate\|Quality Gate]] | Definition of done — 7 gates before declaring shipped |
| [[01-rules/visual-review-non-negotiable\|Visual Review]] | Screenshot + video protocol; overlap gate; scoring |
| [[01-rules/research-first\|Research First]] | scores.md gate; competitive research before any build |
| [[01-rules/skill-invocation-order\|Skill Invocation Order]] | Skill before reasoning; tool before response |
| [[01-rules/skill-self-selection\|Skill Self-Selection]] | Task → skill matrix; auto-invoke rules |
| [[01-rules/image-sourcing-protocol\|Image Sourcing]] | Search order; never silent substitution |
| [[01-rules/demo-to-live-protocol\|Demo-to-Live]] | DEMO tags; CONTENT-NEEDED.md; conversion checklist |
| [[01-rules/client-handoff-protocol\|Client Handoff]] | Handoff package; domain go-live; post-launch |
| [[01-rules/claude-md-rubric\|CLAUDE.md Rubric]] | Iron Laws for project docs; top-1% standard |
| [[01-rules/md-architecture\|MD Architecture]] | How to write rules that actually fire |
| [[01-rules/no-asterisks-in-urls-or-paths\|No Asterisks in URLs]] | Formatting rule — don't break copy-pasted links |

### [[02-categories/website-build-protocol|Categories]]
Platform patterns and deployment archetypes.

| Category | Stack |
|---|---|
| [[02-categories/nextjs-vercel-deploy\|Next.js + Vercel]] | App Router, Fluid Compute, ISR |
| [[02-categories/nextjs-cloudflare-deploy\|Next.js + Cloudflare]] | @opennextjs/cloudflare, D1, R2, KV |
| [[02-categories/nextjs-supabase-saas\|Next.js + Supabase SaaS]] | Auth, RLS, server/client separation |
| [[02-categories/marketing-site\|Marketing Site]] | SEO, CTA patterns, image pipeline |
| [[02-categories/language-learning-app\|Language Learning App]] | TanStack Start, CF Workers, CEFR |
| [[02-categories/medical-practice-site\|Medical Practice Site]] | HIPAA considerations, local SEO |
| [[02-categories/website-build-protocol\|Website Build Protocol]] | Full build pipeline from research to deploy |

### [[03-projects/workspace|Projects]]
Active project bootstrap context.

| Project | URL | Stack |
|---|---|---|
| [[03-projects/workspace\|Workspace Bootstrap]] | — | Global context + decision defaults |
| [[03-projects/block-reign\|Block Reign]] | blockreign.worker-bee.app | Next.js + Sanity CMS |
| [[03-projects/dashboard-worker-bee\|Dashboard Worker Bee]] | dashboard.worker-bee.app | Next.js + Three.js |

### [[00-workspace/workspace-bootstrap|Workspace]]
- [[00-workspace/workspace-bootstrap\|Claude Bootstrap]] — project map, commands, dev tools
- [[00-workspace/vocabulary\|Vocabulary]] — project-specific terms and jargon
- [[00-workspace/failures\|Failure Patterns]] — canonical bugs and lessons

---

> Built with [Quartz](https://quartz.jzhao.xyz) · Updated automatically from `~/.claude`
