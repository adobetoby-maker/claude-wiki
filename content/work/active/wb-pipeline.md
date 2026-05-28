---
ai-first: true
type: project
date: 2026-05-28
tags: [wb-pipeline, droids, agents, pipeline]
status: active
---

# WB Pipeline Droids

## For future Claude
7-phase agent pipeline for building client sites. All droids written as of 2026-05-28. Use when orchestrating a new site build.

---

## Pipeline Map
| Phase | Droid | Path | Input | Output |
|---|---|---|---|---|
| 0 | wb-researcher | `~/.claude/skills/wb-researcher/` | slug | `/tmp/research-brief-<slug>.json` |
| 1 | wb-provisioner | `~/.claude/skills/wb-provisioner/` | research-brief.json | repo + CLAUDE.md |
| 2 | wb-builder | `~/.claude/skills/wb-builder/` | blueprint card order | all cards, 0 TS errors |
| 2.5 | wb-visual-qa | `~/.claude/skills/wb-visual-qa/` | preview URL | `scores.md` |
| 3 | wb-designer | `~/.claude/skills/wb-designer/` | scores.md | all dims ≥8 |
| 4 | wb-qa-gate | `~/.claude/skills/wb-qa-gate/` | staging URL | `qa-report.md` |
| 5 | wb-deployer | `~/.claude/skills/wb-deployer/` | qa-report.md | live URL verified |

## What's Done (as of 2026-05-28)
- All 7 droid markdown files written at `~/.claude/skills/wb-*/agents/*.md`
- All 7 `.claude-plugin/plugin.json` wiring files written
- Memo written: [[brain/how-to-build-droids]]
- Agents tab live at manage.worker-bee.app/sites/[id]/build?tab=agents (XyFlow canvas)

## Invoke a Droid
```bash
# TAC spawns a droid as a subagent:
Agent(subagent_type="wb-researcher-droid", prompt="Research brief for [slug]")
```

## Install All Droids
```bash
for skill in wb-researcher wb-provisioner wb-builder wb-visual-qa wb-designer wb-qa-gate wb-deployer; do
  claude plugin install ~/.claude/skills/$skill
done
```

## Key Design Principles
1. Observable handoffs — file artifact between every phase
2. Narrow expertise — each droid does one phase only
3. Refuse on missing contract — droid exits if predecessor artifact missing
4. Bash-first steps — every Approach step is a command, not a thought
See [[brain/how-to-build-droids]] for full format spec.
