---
ai-first: true
type: north-star
date: 2026-05-28
tags: [identity, goals, strategy]
---

# North Star

## For future Claude
This is the anchor document. Load this first. Every session decision should route through these three pillars.

---

## The Mission
Build a profitable micro-agency that ships client sites at scale through a repeatable 7-phase AI pipeline, while building language-learning products that generate passive income.

## Three Pillars

### 1. WB Agency Pipeline
The [[work/active/manage-worker-bee]] platform is the operating system.
- 7-phase pipeline: Researcher → Provisioner → Builder → Visual QA → Designer → QA Gate → Deployer
- Pipeline droids at `~/.claude/skills/wb-*/` — each phase is a separate agent
- Client sites live at `*.worker-bee.app` subdomains (Cloudflare) or Vercel
- Blueprint canvas at manage.worker-bee.app tracks every site's architecture

### 2. Language Threshold Apps
Passive income from AI-powered language learning:
- [[work/active/medicalspanish]] — medicalspanish.app (live, GA wired)
- [[work/active/constructionspanish]] — constructionspanish.app (live, GA wired)
- Junior Linguist — juniorlinguist.com (auth complete)
- Med Atlas 3D — medterms.worker-bee.app (1,222 cards, Supabase, Stripe)

### 3. Agent Ecosystem (Neural Network)
TAC → Hermes Jr → wba — three coordinated agents:
- TAC (this session): architect, coder, orchestrator
- Hermes Jr: Max OAuth background tasks, vision runs
- wba: long-running daemon, cron jobs, fire-and-forget
- SiteManager: orthobiologic site + LBS Pro orders

## Active Priorities (as of 2026-05-28)
1. DNS cutover: medicalspanish.app + constructionspanish.app → Cloudflare A record `@` → `76.76.21.21`, grey cloud (user must act)
2. Fix claude-wiki git remote — commit `17a1671` local only; push blocked to upstream; see [[log]]
3. WB pipeline droids — all 7 written, plugin.json wired ✓; run `/obsidian-init` to verify vault
4. Vault migration — DONE (14 files, 709 insertions, commit `17a1671`); obsidian-second-brain installed (34 commands)

## What Success Looks Like
- 30+ client sites running through WB pipeline with consistent quality
- Language Threshold apps generating $5K+/mo passive revenue
- Neural network (TAC + Jr + wba) running autonomously overnight
- Every session starts with full context in under 2K tokens
