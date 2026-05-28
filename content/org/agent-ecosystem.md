---
ai-first: true
type: reference
date: 2026-05-28
tags: [agents, hermes, wba, neural-network, tac]
---

# Agent Ecosystem

## For future Claude
The neural network — three coordinated agents. Use this to decide which agent to dispatch for a task.

---

## Agents

### TAC (This Session)
- **Model:** claude-sonnet-4-6
- **Auth:** Claude Code session
- **Role:** Primary architect, coder, orchestrator
- **Use for:** TypeScript architecture, multi-file debugging, agent orchestration, content writing

### Hermes Jr
- **Command:** `jr "task"`
- **Model:** claude-sonnet-4-6 via Max OAuth
- **Auth:** macOS Keychain "Claude Code-credentials" (same OAuth token as TAC)
- **Output:** Teed to `/tmp/jr-YYYYMMDD-HHMMSS.txt`
- **Use for:** Synchronous background tasks where TAC needs the output
- **CRITICAL:** Always use `Bash(timeout=600000)` + `jr "task"` — NEVER `run_in_background=True` when TAC needs results

### Worker Bee Agent (wba)
- **Command:** `wba "task"` (inline) / `wba -b "task"` (background queue)
- **Daemon:** `~/.worker-bee/daemon.py`
- **Auth:** Max OAuth via `claude -p`
- **Use for:** Long-running tasks, cron jobs, fire-and-forget work
- **Cron jobs:** site-monitor (30m), memory-daily-sync (7am), willie-elam-social (Mon 8am, DISABLED)

### SiteManager
- **Command:** `hermes --profile sitemanager -z "task"`
- **Use for:** orthobiologic site ONLY, LBS Pro orders

### Dispatch
- **Command:** `dispatch --bg "task"` (fire-and-forget via Hermes)
- **Logs:** `~/.hermes/logs/`

## Dispatch Decision Matrix
| Task type | Agent | Command |
|---|---|---|
| TAC needs output to act on | Jr | `Bash(timeout=600000)` + `jr "task"` |
| Vision run (screenshot analysis) | Jr | same — synchronous |
| True fire-and-forget | wba | `wba -b "task"` |
| Long-running + persist after session | wba daemon | `wba -b "task"` |
| Orthobiologic/LBS | SiteManager | `hermes --profile sitemanager` |
| Haiku mechanical work | jr with model flag | `jr -m haiku "task"` |

## Identity Files
- `~/.claude/SOUL.md` — TAC identity
- `~/.claude/AGENTS.md` — full ecosystem map
- `~/.hermes/SOUL.md` — Hermes identity
- `~/.hermes-jr/SOUL.md` — Jr identity
- `~/.hermes-jr/profiles/*/` — personality SOULs

## Failure Pattern
- Using `hermes-jr -z "task" &` directly → bypasses `jr` wrapper, no auto-log, output lost
- Using `Bash(run_in_background=True)` with `jr` when TAC needs the result → TAC is blind
