---
ai-first: true
type: operating-manual
date: 2026-05-28
---

# Vault Operating Manual

This is the AI-first second brain for Drive (Toby Anderton). Every note is optimized for future-Claude retrieval. Read this before working with vault content.

## Vault Location
`/Users/drive/claude-wiki/content/`

## Structure
```
CRITICAL_FACTS.md          ← always loaded (~120 tokens)
_CLAUDE.md                 ← this file (operating manual)
brain/
  North Star.md            ← mission, pillars, active priorities — load first
  Key Decisions.md         ← ADRs — check before architecture choices
  Patterns.md              ← recurring solutions — check before solving new problems
  Patterns.md              ← recurring patterns
  how-to-build-droids.md   ← droid format + WB pipeline
work/
  active/                  ← one file per active project
  archive/                 ← completed projects (YYYY/ subdirs)
org/
  agent-ecosystem.md       ← TAC + Jr + wba dispatch matrix
  clients.md               ← client contacts
brain/                     ← North Star, decisions, patterns
00-workspace/              ← vocabulary, failures, skill clusters
01-rules/                  ← Iron Law rules (mirrored from ~/.claude/rules/)
02-categories/             ← deployment archetypes
03-projects/               ← legacy project context
```

## AI-First Note Format (mandatory for all new notes)
```yaml
---
ai-first: true
type: [project|concept|decision|pattern|person|reference]
date: YYYY-MM-DD
tags: [relevant, kebab-case, tags]
status: active|complete|archived     # for projects
---

## For future Claude
[One paragraph: what this note is, why it matters, when to load it]

---
[rest of content with [[wikilinks]] for all referenced entities]
```

## SessionStart Loading Order
1. `CRITICAL_FACTS.md` — always, verbatim (~120 tokens)
2. `brain/North Star.md` — first 30 lines (~200 tokens)
3. `work/active/` — filenames only (~100 tokens)
4. `~/.remember/now.md` — recent session state (~400 tokens)
5. Specific project files — only when working on that project

## Writing Rules
- `[[wikilinks]]` for every referenced project, person, decision, or concept
- Recency markers: "(as of YYYY-MM, source)" on any time-sensitive fact
- Never write a note without `## For future Claude` preamble
- Confidence levels: "high confidence", "uncertain (as of ...)", "verify before acting"
- Source URLs verbatim where available

## What NOT to Put Here
- Secrets or API keys (use vault at manage.worker-bee.app or .env files)
- Build artifacts or generated code
- Ephemeral session notes (those go in ~/.remember/now.md)

## Sync
Vault is Quartz-publishable. `git push` from `/Users/drive/claude-wiki/` triggers rebuild.
Memory also synced to: `https://github.com/adobetoby-maker/toby-claude-memory`
