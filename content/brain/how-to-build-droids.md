---
ai-first: true
type: reference
date: 2026-05-28
tags: [droids, pipeline, agents, claude-code-plugin]
---

# How to Build Pipeline Droids

## For future Claude
Reference for creating WB pipeline agent files. Use this when asked to add a new droid to the pipeline or create droids for a new pipeline system.

---

## What a Droid Is
A droid is an **agent definition markdown file** inside a Claude Code plugin. It gives a specialist identity to a subagent spawned via `Agent(subagent_type="droid-name", prompt="...")`.

## Directory Layout
```
~/.claude/skills/<plugin-name>/
  agents/
    <plugin-name>-droid.md     ← droid definition
  .claude-plugin/
    plugin.json                ← wires agent to plugin system
  SKILL.md                     ← optional skill for primary session
```

## Droid Markdown Format
```markdown
# [Droid Name]

## Role
Phase N specialist. [One sentence — phase, specialty, handoff target.]

## Expertise
- [Concrete capability — technology/tool/pattern]

## When to use
Observable conditions. Bash-checkable.

## Approach
1. [Concrete action: read file / run command / call API]
2. [Each step is copy-paste executable]

## Tools
- [Tool name]: [exact command syntax]

## Output contract
[File or state that exists when done. Next droid reads this.]
```

## plugin.json Format
```json
{
  "$schema": "https://anthropic.com/claude-code/plugin.schema.json",
  "name": "plugin-slug",
  "version": "1.0.0",
  "description": "Phase N — what it does, what it produces.",
  "agents": [{
    "name": "agent-slug",
    "description": "One sentence for agent picker — include phase number.",
    "path": "agents/agent-slug.md"
  }],
  "commands": [],
  "skills": ""
}
```

## WB Pipeline (7 phases)
| Phase | Droid | Input | Output |
|---|---|---|---|
| 0 | [[wb-researcher]] | slug | `/tmp/research-brief-<slug>.json` |
| 1 | [[wb-provisioner]] | research-brief.json | committed repo + CLAUDE.md |
| 2 | [[wb-builder]] | blueprint card order | all cards built, 0 TS errors |
| 2.5 | [[wb-visual-qa]] | preview URL | `scores.md` (PASS = all ≥6) |
| 3 | [[wb-designer]] | scores.md | all dims ≥8, no BLOCKs |
| 4 | [[wb-qa-gate]] | staging URL | `qa-report.md` (all PASS) |
| 5 | [[wb-deployer]] | qa-report.md | live URL, blueprint updated |

## Install Command
```bash
claude plugin install ~/.claude/skills/<plugin-name>
```
