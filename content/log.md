---
ai-first: true
type: log
date: 2026-05-28
tags: [log, operations, sessions, migrations]
---

# Operation Log

## For future Claude
Chronological record of significant vault operations, migrations, and infrastructure changes. Load when you need to understand what changed and when. Not for individual project work — see work/active/ for that.

---

## 2026-05-28 — Vault Migration + obsidian-second-brain Install

**Trigger:** Session continuation after context compaction; user requested Obsidian community tool research then said "ok go to work lets get it all done."

**Community tools researched:**
- `obsidian-mind` (breferrari) — vault template, tiered SessionStart loading, AI-first note format, 18 commands
- `kepano/obsidian-skills` — Agent Skills spec; SKILL.md with `name` + `description` frontmatter; 5 skills: obsidian-markdown, obsidian-bases, json-canvas, obsidian-cli, defuddle
- `obsidian-second-brain` (eugeniughelbur) — 34 commands, self-updating vault, research toolkit (x-read, x-pulse, research, research-deep, notebooklm, youtube), 4 scheduled background agents

**Decision made:** Adopt obsidian-mind AI-first format as mandatory vault standard. Migrate from flat `~/.claude/projects/-Users-drive/memory/` to this vault. Install obsidian-second-brain for research toolkit.

**Files created in vault:**
- `CRITICAL_FACTS.md` — always-loaded constants (~120 tokens)
- `_CLAUDE.md` — vault operating manual
- `brain/North Star.md` — mission + active priorities
- `brain/Key Decisions.md` — ADRs
- `brain/Patterns.md` — recurring solutions
- `brain/how-to-build-droids.md` — droid format memo
- `org/agent-ecosystem.md` — TAC + Jr + wba dispatch matrix
- `work/active/manage-worker-bee.md`
- `work/active/jrs-auto-repair.md`
- `work/active/language-threshold.md`
- `work/active/wb-pipeline.md`
- `index.md` (modified)

**Commit:** `17a1671` — 14 files changed, 709 insertions, branch `v4`

**obsidian-second-brain installed:**
- 34 commands registered in `~/.claude/commands/`
- Vault path configured in `~/.claude/settings.json`
- PostCompact + SessionStart hooks wired by `setup.sh`
- Install: `bash ~/.claude/skills/obsidian-second-brain/scripts/setup.sh "/Users/drive/claude-wiki/content"`

**SessionStart hook upgraded:**
- Path: `/Users/drive/.claude/bootstrap/session-start.sh`
- Strategy: vault-first tiered loading (CRITICAL_FACTS → North Star excerpt → active filenames → recent state)
- Old: loaded entire MEMORY.md index flat

**SRI exception documented:**
- Google's `gtag.js` cannot use SRI integrity hash — Google dynamically updates the script; any fixed hash breaks it
- Security hook at PostToolUse:Write correctly flags external scripts but gtag.js is a known exception
- No `integrity` attribute on gtag.js script tags is correct

**Git push BLOCKED:**
- Remote: `jackyzha0/quartz.git` (upstream Quartz template, not user fork)
- Branch: `v4`
- `adobetoby-maker` has no push access to upstream
- Commit `17a1671` is local only

**Pending (as of 2026-05-28):**
- [ ] Fix git remote — user must create fork or use `toby-claude-memory` as remote, then `git push`
- [ ] DNS cutover — Cloudflare dashboard: A record `@` → `76.76.21.21`, grey cloud, for medicalspanish.app + constructionspanish.app
- [ ] Run `/obsidian-init` — Claude scans vault and verifies operating manual
- [ ] Optional MCP: `claude mcp add obsidian-vault -s user -- npx -y mcp-obsidian "/Users/drive/claude-wiki/content"`
- [ ] Migrate remaining ~25 project files from `~/.claude/projects/-Users-drive/memory/` to vault
