---
ai-first: true
type: reference
date: 2026-05-28
tags: [wba, worker-bee-agent, daemon, max-oauth, agent]
---

# Worker Bee Agent (wba)

## For future Claude
Max OAuth autonomous agent — no API key, no billing, runs on Claude Max subscription via `claude -p`. Load this when dispatching background tasks, checking cron jobs, or debugging wba failures.

---

## What It Is
wba is a task queue + daemon that uses `claude -p` (Claude Code CLI, non-interactive) as its execution engine. Max OAuth token works in Claude Code session context — Hermes silently fails with the same token because it makes direct HTTP calls to api.anthropic.com (scope mismatch: `user:sessions:claude_code` ≠ direct API scope).

## Files
| Path | Purpose |
|---|---|
| `~/.worker-bee/daemon.py` | Main daemon (~420 lines) |
| `~/.worker-bee/bin/wba` | CLI shim → `exec python3 ~/.worker-bee/daemon.py "$@"` |
| `~/.worker-bee/cron/jobs.json` | Scheduled cron job definitions |
| `~/.worker-bee/tasks/pending/` | Task queue (pending → running → done/failed) |
| `~/.worker-bee/logs/` | Execution logs |
| `~/.worker-bee/skills/` | Skill cache |

PATH wired in `~/.zshrc`: `export PATH="$HOME/.worker-bee/bin:$PATH"`

## Commands
| Command | What it does |
|---|---|
| `wba "task"` | Inline — runs now, blocks, prints result |
| `wba -b "task"` | Background — queues to daemon, returns immediately |
| `wba -s skill-name "task"` | Load skill context before running task |
| `wba -m haiku "task"` | Force model: haiku / sonnet / opus |
| `wba -n "task"` | iMessage notification when done |
| `wba start / stop / status` | Daemon lifecycle |
| `wba cron` | List scheduled jobs |

## Active Cron Jobs
- `site-monitor` — every 30 min, haiku, checks worker-bee site health → iMessage
- `memory-daily-sync` — 7am daily, haiku, git pull memory repo
- `willie-elam-social` — Mondays 8am, sonnet — DISABLED

## Architecture
```
wba "task"
  → queues JSON to ~/.worker-bee/tasks/pending/
  → daemon.py polls queue
  → builds prompt (task + optional skill context)
  → subprocess: claude -p "<prompt>" --output-format text
  → result written to ~/.worker-bee/tasks/done/
  → logs to ~/.worker-bee/logs/
```

## Why Not Hermes
Hermes calls api.anthropic.com directly. The Max OAuth token has scope `user:sessions:claude_code` — valid only inside Claude Code session context, not raw HTTP. Hermes silently marks the token exhausted on every call. Three paths:
- Hermes + OAuth token → silent fail (scope mismatch)
- Hermes + API key → works (uses API credits)
- wba + `claude -p` → works (correct session context, Max OAuth)

## Failure Modes
- `wba status` shows daemon not running → `wba start`
- Task stuck in pending/ → check logs/, may be a claude -p auth issue
- `claude -p` not found → ensure Claude Code CLI is in PATH
- Using `hermes-jr -z "task" &` directly → bypasses jr wrapper, no auto-log, output lost
