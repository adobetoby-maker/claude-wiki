---
title: Agent Ecosystem
tags: [agents, architecture, hermes, wba, qwen]
---

# Agent Ecosystem — Drive's Neural Network

Five interconnected agents share one OAuth token, one filesystem, and one memory store. Each owns a distinct execution path.

---

## Topology Overview

```
macOS Keychain "Claude Code-credentials"
         │  (one OAuth token, shared)
         ├──▶ Claude Code (TAC)        — direct SDK
         ├──▶ Hermes Jr                — claude -p subprocess
         ├──▶ wba daemon               — claude -p subprocess
         └──▶ Hermes                   — anthropic_adapter.py reads keychain
                   └──▶ SiteManager   — hermes --profile sitemanager

Qwen 3.6-27B  —  separate local inference, no Anthropic key
         └──▶ Hermes local profile     — hermes --profile local
         └──▶ tac-hermes() shell fn
```

---

## Agent 1: Claude Code / TAC

| Field | Value |
|---|---|
| Role | Primary architect, coder, session orchestrator |
| Model | `claude-sonnet-4-6` default / `claude-opus-4-7` high-stakes |
| Auth | macOS Keychain `"Claude Code-credentials"` → sk-ant-oat01-* |
| Identity | `~/.claude/SOUL.md` |
| Bootstrap | `/tac` skill — syncs memory, shows projects, model routing, skills menu |
| Memory | `~/.claude/projects/-Users-drive/memory/` (git-backed) |
| Invoke | `tac [project]` → `claude --dangerously-skip-permissions` |

**Strengths:** Multi-file reasoning, architecture, tool orchestration, MCP access, research.

**Key files:**
- `~/.claude/CLAUDE.md` — workspace rules
- `~/.claude/AGENTS.md` — full agent map
- `~/.claude/SOUL.md` — identity
- `~/.claude/rules/*.md` — 10 iron-law rule files
- `~/.claude/skills/tac/SKILL.md` — session bootstrap skill
- `~/.claude/hooks.json` — pre/post tool hooks

---

## Agent 2: Hermes Jr

| Field | Value |
|---|---|
| Role | Full Hermes interface powered by `claude -p` — zero API billing |
| Model | `claude-sonnet-4-6` via Max OAuth |
| Auth | Max OAuth via Claude Code CLI — `user:sessions:claude_code` scope |
| Identity | `~/.hermes-jr/SOUL.md` |
| Profiles | `~/.hermes-jr/profiles/claude/` · `~/.hermes-jr/profiles/sitemanager/` |
| Repo | `~/hermes-jr-agent/` → branch `hermes-jr` |
| Invoke | `jr "task"` · `jrs "task"` · `hermes-jr --profile claude -z "task"` |

**Why it exists:** Hermes original uses `api.anthropic.com` directly — Max OAuth's `user:sessions:claude_code` scope fails there. Hermes Jr routes through `claude -p` which works perfectly.

**Key files:**
- `~/hermes-jr-agent/agent/claude_p_executor.py` — 191-line core (replaces 800-line anthropic_adapter)
- `~/hermes-jr-agent/hermes_jr_main.py` — 36-line entry point
- `~/hermes-jr-agent/hermes_cli/oneshot.py` — `_run_agent()` rewritten to 7 lines

**Shell aliases (`.zshrc`):**
```bash
jr "task"                    # oneshot, claude profile
jr -p teacher "task"         # with personality flag
jrs "task"                   # oneshot, sitemanager profile
hermes-jr --profile claude   # interactive chat
```

**Personalities:** teacher · concise · technical · philosopher · hype · pirate

---

## Agent 3: wba (Worker-Bee Agent Daemon)

| Field | Value |
|---|---|
| Role | Lightweight autonomous daemon — background queue, cron |
| Model | `claude-sonnet-4-6` default / `claude-haiku-4-5-20251001` for cron |
| Auth | Max OAuth via Claude Code CLI — same Keychain token |
| Source | `~/.worker-bee/daemon.py` (~420 lines, written from scratch) |
| Queue | `~/.worker-bee/tasks/` (pending → running → done/failed) |
| Cron | `~/.worker-bee/cron/jobs.json` (5-field cron, 60s dedup window) |
| Invoke | `wba "task"` · `wba -b "task"` · `wba start` |

**Key difference from Hermes:** ~420 lines vs ~5000 lines. `claude -p` subprocess, no credential pool, no API key exhaustion tracking.

```bash
wba "task"          # inline, blocks, prints result
wba -b "task"       # background queue dispatch
wba -s skill "task" # inject skill context before task
wba -m haiku "task" # force model
wba -n "task"       # notify via iMessage when done
wba start / stop / status  # daemon lifecycle
wba cron            # list scheduled jobs
```

---

## Agent 4: Hermes (Original)

| Field | Value |
|---|---|
| Role | Persistent gateway agent, cron, iMessage/Telegram/Slack |
| Model | `claude-sonnet-4-6` via Claude OAuth |
| Auth | `anthropic_adapter.py` reads Keychain — same token |
| Identity | `~/.hermes/SOUL.md` |
| Memory | `~/.hermes/memory_store.db` (SQLite FTS5) + team-memory |
| Skills | `~/.hermes/skills/` |
| Invoke | `hermes --profile claude -z "task"` · `hermes chat` |

**Active cron jobs:**
| Name | Schedule | What |
|---|---|---|
| `site-monitor` | every 30m | Checks sites, iMessage alert |
| `lbs-daily-sync` | 6am daily | LBS inventory → Supabase |
| `willie-elam-social-post` | Mon 8am | Social post drafts |
| `memory-daily-sync` | 7am daily | Git pull memory |

---

## Agent 5: SiteManager

| Field | Value |
|---|---|
| Role | Orthobiologic site manager — inventory, orders, monitoring |
| Model | `claude-sonnet-4-6` via Claude OAuth |
| Sites | orthobiologicpathways.com · ime-coach.com |
| Supplier | LBS Pro Advanced |
| Invoke | `hermes --profile sitemanager -z "sync inventory"` · `jrs "task"` |
| Memory | `~/.hermes/profiles/sitemanager/memories/` |

---

## Agent 6: Qwen 3.6-27B (Local)

| Field | Value |
|---|---|
| Role | Zero-API-cost local inference, vision, overnight tasks |
| Model | Qwen3.6-27B-UD-Q4_K_XL (27B params, 4-bit quant) |
| Hardware | Apple M1 Ultra — 110GB unified memory |
| Context | 131072 tokens (`--parallel 1` — full slot) |
| Port | `:8090` (llama-server) |
| Auto-start | `~/Library/LaunchAgents/com.drive.qwen-server.plist` |
| Invoke | `tac-hermes` · `hl "task"` · `hermes --profile local -z "task"` |

**Server flags:** `--n-gpu-layers 999 --flash-attn on --reasoning-format deepseek --cache-type-k q4_0 --cache-type-v q4_0 --parallel 1 --no-mmap`

**Why `--parallel 1`:** Multiple slots divide `ctx-size` evenly. With `--parallel 4` (old default): 131072 ÷ 4 = 32768 per slot. With `--parallel 1`: full 131072. Hermes requires 64K minimum.

```bash
tac-hermes          # start server if needed, drop into interactive chat
tac-hermes "task"   # one-shot
hl "task"           # alias: hermes --profile local -z
```

---

## Memory Architecture

| Store | Location | Used by |
|---|---|---|
| Git-backed flat files | `~/.claude/projects/-Users-drive/memory/` | TAC primary |
| SQLite FTS5 | `~/.hermes/memory_store.db` | Hermes primary |
| Shared team memory | `~/.hermes/team-memory/shared_memory.db` | Both |
| AgentDB HNSW | RAM / ruflo-agentdb | Both via ruflo bridge |

**Memory CRUD from terminal:**
```bash
mem-store KEY "value" [ns]   # store
mem-get KEY [ns]             # retrieve
mem-search "query" [ns]      # semantic search
mem-list [ns]                # list recent
ruflo sync                   # pull GitHub → AgentDB
```

---

## Dispatch Patterns

### 1. TAC → Background (Hermes)
```bash
hermes --profile claude -z "task" &
```

### 2. TAC → wba queue
```bash
wba -b "task"
wba -b -n "task"   # + iMessage notify when done
```

### 3. iMessage gateway
Text your Mac → Hermes wakes → acts → texts back.

### 4. Cron (Hermes)
```bash
# Edit: ~/.hermes/cron/jobs.json
# List: hermes cron list
# Status: hermes cron status
```

### 5. Qwen overnight
```bash
overnight-build   # runs overnight-builder.sh
```

---

## Adding a New Agent

1. Clone or create at `~/agent-name/`
2. Write `~/.hermes/profiles/<name>/SOUL.md` — identity + mission
3. Copy nearest `config.yaml`, update model/provider/context_length
4. Add shell alias to `.zshrc`
5. Add row to this page + to `~/.claude/AGENTS.md`
6. Test: `hermes --profile <name> -z "introduce yourself"`
