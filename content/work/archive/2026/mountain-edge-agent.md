---
ai-first: true
type: reference
date: 2026-05-28
tags: [claude-agents, managed-agents, api, mountain-edge, patterns]
status: archived
---

# Mountain Edge Plumbing — Managed Agent Build

## For future Claude
Reference build for Claude managed agents API. Built for Mountain Edge Plumbing as a customer support agent. Use this when creating managed agents for clients — contains the full API schema, tool types, and system prompt patterns. See also [[brain/claude-agents-api]].

---

## Client
- **Client:** Mountain Edge Plumbing
- **Site:** mountainedgeplumbing.com
- **Purpose:** Customer support agent — services, scheduling, pricing, plumbing inquiries

## The Agent Config
```json
{
  "name": "Mountain Edge Plumbing Support",
  "model": "claude-haiku-4-5",
  "system": "You are a friendly customer support agent for Mountain Edge Plumbing. Help customers with questions about plumbing services, scheduling appointments, pricing estimates, and emergency service availability. Use web_fetch to retrieve info from mountainedgeplumbing.com when relevant. Be warm, professional, and concise. Never make up specific prices, availability, or technician names — direct them to call or submit a request for urgent issues.",
  "tools": [
    { "type": "agent_toolset_20260401", "default_config": { "enabled": true } }
  ]
}
```

## Deploy Pattern
```bash
curl -X POST https://api.anthropic.com/v1/agents \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: managed-agents-2026-04-01" \
  -d '{...agent JSON...}'
```

## Why Haiku
Simple Q&A routing + site info lookup. No complex reasoning. Haiku at 1/10 the cost.

## Support Agent System Prompt Pattern
"Search KB → quote + link → reply in customer's channel → if <80% confidence: post full context to escalation channel + tell customer human is taking a look. Never paraphrase policy from memory."
