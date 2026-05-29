---
ai-first: true
type: reference
date: 2026-05-28
tags: [claude, anthropic, managed-agents, api, patterns]
---

# Claude Managed Agents API

## For future Claude
API schema and patterns for creating Claude managed agents via `/v1/agents`. Beta feature (as of 2026-04). Load this when building client-facing agents, support bots, or any Anthropic managed agent. See also [[work/archive/2026/mountain-edge-agent]] for a real build example.

---

## Endpoint
```
POST https://api.anthropic.com/v1/agents
Headers:
  anthropic-beta: managed-agents-2026-04-01   ← required
  x-api-key: $ANTHROPIC_API_KEY
  anthropic-version: 2023-06-01
```

## Agent JSON Schema
```json
{
  "name": "string",
  "description": "string",
  "model": "claude-opus-4-7 | claude-sonnet-4-6 | claude-haiku-4-5",
  "system": "string — full system prompt",
  "mcp_servers": [
    { "name": "notion", "type": "url", "url": "https://mcp.notion.com/mcp" },
    { "name": "slack", "type": "url", "url": "https://mcp.slack.com/mcp" }
  ],
  "tools": [
    { "type": "agent_toolset_20260401", "default_config": { "enabled": true } },
    {
      "type": "mcp_toolset",
      "mcp_server_name": "notion",
      "default_config": { "permission_policy": { "type": "always_allow" } }
    }
  ]
}
```

## Tool Types
| Type | Purpose |
|---|---|
| `agent_toolset_20260401` | Memory, sessions, sub-agent spawning |
| `mcp_toolset` | Connects named MCP server; `always_allow` = no per-call approval |

## Model Selection
- `claude-haiku-4-5` — simple Q&A, routing, support (1/10 cost)
- `claude-sonnet-4-6` — reasoning + tool use (default)
- `claude-opus-4-7` — complex multi-step agentic tasks (10x cost)

## Support Agent System Prompt Pattern
```
1. Search KB → quote + link → never paraphrase policy from memory
2. Reply in customer's channel: direct answer → source link → one proactive next step
3. If <80% confidence: post full context to escalation channel + tell customer a human is on it
Match customer's tone. Warm but don't pad. One emoji max.
```

## Known MCP Server URLs
| Service | URL |
|---|---|
| Notion | https://mcp.notion.com/mcp |
| Slack | https://mcp.slack.com/mcp |
