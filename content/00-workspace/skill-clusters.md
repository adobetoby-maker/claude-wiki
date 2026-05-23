---
title: Skill Clusters by Model
tags: [skills, models, routing, haiku, sonnet, opus, qwen]
---

# Skill Clusters — Which Model Uses What

Skills are routed to models by cost and capability. Wrong routing wastes money; right routing makes the work faster.

---

## The Prime Rule

```
"Do X to Y files"  →  Haiku   (~1/10 cost)
"Design X system"  →  Sonnet  (default)
"Decide strategy"  →  Opus    (use sparingly — 10× cost)
"Run overnight"    →  Qwen    (zero API cost)
```

In Claude Code: `Agent(model="haiku", ...)` for mechanical subagents.

---

## Haiku Cluster — Mechanical Operations

**Model:** `claude-haiku-4-5-20251001`
**Cost:** ~1/10 of Sonnet
**When:** "Just do X" tasks with no ambiguity

| Skill / Task | Command |
|---|---|
| git add, commit, push | `git commit -m "..."` |
| npm install / lint / build | `npm install`, `npm run lint` |
| File rename / mv / cp | direct bash |
| curl download | `curl -o ...` |
| Image resize / format convert | `sips`, `ffmpeg` |
| String replacements across files | `sed`, Edit tool |
| Image download | `curl` |
| Env var lookup | `grep -r VAR .env*` |

**Spawn pattern:**
```typescript
Agent({ model: "haiku", prompt: "rename all .jpeg to .jpg in /public/images" })
```

---

## Sonnet Cluster — Default Development

**Model:** `claude-sonnet-4-6`
**Cost:** Base rate
**When:** All architecture, debugging, writing, and orchestration

### Frontend Sub-cluster (Antigravity)
| Skill | Invoke | What |
|---|---|---|
| Next.js best practices | `/nextjs-best-practices` | App Router, RSC, caching |
| App Router patterns | `/nextjs-app-router-patterns` | Streaming, PPR, layouts |
| React 19 | `/react-best-practices` | Memoization, transitions |
| shadcn/ui | `/shadcn` | Component management |
| Tailwind design system | `/tailwind-design-system` | Production tokens |
| Tailwind v4 | `/tailwind-patterns` | CSS-first config |
| Landing page | `/landing-page-generator` | High-converting pages |
| Auth | `/nextjs-supabase-auth` | Supabase + Next.js auth |

### Platform Sub-cluster (Antigravity)
| Skill | Invoke | What |
|---|---|---|
| Supabase | `/supabase-automation` | DB, tables, RLS, admin |
| Cloudflare Workers | `/cloudflare-workers-expert` | Workers, D1, R2, KV |
| Vercel AI SDK | `/vercel-ai-sdk-expert` | Streaming, tools, RAG |
| Vercel deploy | `/vercel-deployment` | Project setup, env |
| Vercel automation | `/vercel-automation` | CI/CD, project mgmt |

### SEO Sub-cluster (Antigravity)
| Skill | Invoke | What |
|---|---|---|
| AEO blog writer | `/seo-aeo-blog-writer` | Long-form SEO posts |
| SEO audit | `/seo-audit` | Crawl/index diagnostics |
| Technical SEO | `/seo-technical` | Per-page metadata audit |
| Keyword strategist | `/seo-keyword-strategist` | Density + analysis |
| Content strategy | `/content-strategy` | Topic clusters + roadmap |
| Copywriting | `/copywriting` | Conversion-focused copy |
| Keyword extractor | `/keyword-extractor` | Keyword mining |

### GStack Sub-cluster
| Skill | Invoke | What |
|---|---|---|
| Code review | `/review` | Staff engineer lens |
| QA | `/qa` | Browser QA, edge cases |
| Ship | `/ship` | PR + merge automation |
| Security | `/cso` | OWASP + STRIDE audit |
| Debug | `/investigate` | Deep-dive debugging |
| Auto-plan | `/autoplan` | Plan before coding |
| Design shotgun | `/design-shotgun` | 3 UI directions fast |
| Context save | `/context-save` | Save session to memory |
| Context restore | `/context-restore` | Restore saved session |
| Browse | `/browse` | Web browsing |
| Benchmark | `/benchmark` | Performance benchmarking |
| Retro | `/retro` | Post-launch retrospective |
| Guard | `/guard` | Protect critical code |

### Architecture Sub-cluster (Antigravity)
| Skill | Invoke | What |
|---|---|---|
| Multi-agent patterns | `/multi-agent-patterns` | System design |
| Parallel agents | `/parallel-agents` | Orchestration patterns |
| Production audit | `/production-code-audit` | Deep codebase scan |
| Prompt engineering | `/prompt-engineering` | Prompt design |
| GitHub Actions | `/github-actions-templates` | CI/CD workflows |
| API design | `/api-design-principles` | REST/GraphQL |
| Testing patterns | `/testing-patterns` | Jest + factories |
| E2E testing | `/e2e-testing-patterns` | E2E test suite |

---

## Opus Cluster — Strategic Decisions Only

**Model:** `claude-opus-4-7`
**Cost:** 10× Sonnet — use sparingly
**When:** Product decisions, high-stakes architecture, anything where being wrong is expensive

| Task | Notes |
|---|---|
| `/office-hours` | CEO product/strategy session |
| `/plan-eng-review` | Architecture lockdown |
| `/plan-ceo-review` | Founder feature review |
| Pricing / positioning | Business model decisions |
| "Which architecture for the whole system" | Not "which component" |

**Rule:** If you can solve it with Sonnet, do. Reserve Opus for decisions that can't be undone cheaply.

---

## Qwen / Local Cluster — Zero API Cost

**Model:** Qwen3.6-27B running locally via llama-server on `:8090`
**Cost:** $0 (runs on M1 Ultra)
**When:** Overnight builds, long reasoning tasks, vision tasks, anything you'd feel bad spending API credits on

| Task | Command |
|---|---|
| Interactive local chat | `tac-hermes` |
| One-shot task | `hl "task"` |
| Overnight build | `overnight-build` |
| Overnight dry run | `overnight-dry` |
| Vision / image analysis | Built-in (mmproj-F16.gguf) |
| Long context reasoning | 131k token window |

**Context window:** 131072 tokens (`--parallel 1`). The model is trained on 262144 but the server caps at 131k (sufficient for very large codebases).

**Auto-starts on login** via `~/Library/LaunchAgents/com.drive.qwen-server.plist`.

---

## Ruflo Suite — Advanced Agent Capabilities

**Model:** Sonnet (default) / Opus for strategic tasks
**Access:** Via skill invocation or Agent tool

| Skill | Invoke | What |
|---|---|---|
| RAG memory | `/ruflo-rag-memory:ruflo-memory` | Vector store + semantic search |
| Cost tracker | `/ruflo-cost-tracker:ruflo-cost` | Token cost tracking |
| SPARC | `/ruflo-sparc:ruflo-sparc` | 5-phase build methodology |
| Swarm coordinator | `Agent(subagent_type="ruflo-swarm:coordinator")` | Multi-agent orchestration |
| GOAP planner | `Agent(subagent_type="ruflo-goals:goal-planner")` | A* action planning |
| Deep researcher | `Agent(subagent_type="ruflo-goals:deep-researcher")` | Multi-source research |
| Security auditor | `Agent(subagent_type="ruflo-security-audit:security-auditor")` | Vulnerability scan |
| TDD tester | `Agent(subagent_type="ruflo-testgen:tester")` | TDD London School |

---

## Decision Matrix — Which Model for Which Task?

| Situation | Model | Why |
|---|---|---|
| "Rename all these files" | Haiku | Mechanical — no reasoning needed |
| "Add dark mode to this component" | Haiku | Single-file, straightforward |
| "Build the checkout flow" | Sonnet | Multi-file, architecture decisions |
| "Debug this auth race condition" | Sonnet | Multi-file reasoning |
| "Write SEO blog post" | Sonnet | Content quality matters |
| "Decide between Next.js and Remix for this project" | Opus | Strategic, affects whole project |
| "What's our monetization strategy?" | Opus | Business-critical |
| "Run overnight link audit across 200 pages" | Qwen | Long task, zero cost |
| "Analyze this screenshot" | Qwen | Vision, zero cost |

---

## Per-Agent Skill Access

| Agent | Can use | Typical skills |
|---|---|---|
| Claude Code (TAC) | All skills (MCP + Skill tool) | All clusters above |
| Hermes Jr | Skills injected via SOUL.md context | GStack, platform basics |
| wba daemon | Skills via `-s skill-name` flag | Mechanical + content |
| Hermes (original) | `~/.hermes/skills/` registry | autonomous-ai-agents, software-dev |
| Qwen (via Hermes local) | Same as Hermes, runs locally | General reasoning, vision |

---

## Model Routing in Practice (from `.zshrc`)

```bash
export ANTHROPIC_MODEL="claude-sonnet-4-6"   # global default

# Force model on any one-shot:
jr "task"                     # Sonnet via Max OAuth
wba -m haiku "rename files"   # Haiku via Max OAuth
wba -m sonnet "build feature" # Sonnet

# Inside Claude Code session:
# Agent(model="haiku", ...)   # spawn Haiku subagent
# /model haiku                # switch current session
```
