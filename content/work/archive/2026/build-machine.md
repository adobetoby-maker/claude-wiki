---
ai-first: true
type: project
date: 2026-05-28
tags: [build-machine, physician-sites, pipeline, worker-bee, archived]
status: archived
---

# Build Machine — Physician Site Pipeline

## For future Claude
The original 10-build loop for identity-differentiated physician marketing sites. Built 8/10 before the WB pipeline replaced it. Load this to understand the build-api pattern, iteration modes, and deployed physician sites.

---

## What It Was
10-build loop: blueprint-wizard → build-api → claude CLI → identity-differentiated physician marketing sites. Each doctor gets a unique palette + typography to avoid identical sites.

## Build-API Call Pattern
```bash
POST build-api.worker-bee.app/run
x-api-key: wb-build-local-9f4a2c
Body: { buildMode: "new" | "iteration", localPath, ... }
```
- `buildMode: "iteration"` — skips scaffold, claude rewrites not patches
- `localPath` derived from `github_repo.split('/')[1]` — NOT `toSlug(name)` (honorifics break it)

## Deployed Physician Sites
| Site | Domain | Repo |
|---|---|---|
| Dr. Guy Grooms MD | guygrooms-md.worker-bee.app | adobetoby-maker/guy-grooms-md |
| Dr. Kyle Nay DPM | kylenaydpm.worker-bee.app | adobetoby-maker/kyle-nay-dpm |
| Toby Anderton MD | tobyandertonmd.com | adobetoby-maker/toby-anderton-s-site |

## Supabase Site IDs (manage-worker-bee)
- Dr. Guy Grooms MD: `bc8df2e9-020b-4039-93c8-28d7609a04d7`
- Dr. Kyle Nay DPM: `bbec57ef-9e57-4a44-80ee-d64da4f7eb88`
- Toby Anderton: `d17b487c-7b4c-4844-8daa-73bd6d1e6b37`

## Identity Palette System
- Grooms: Marine red `#c41e3a` — military precision, sports medicine
- Nay: Forest green `#1a3a2e` + Amber `#e8a020` — Idaho outdoorsman, active patients
- Anderton: Midnight charcoal `#1c1c2e` + Steel blue `#2563eb` + Gold `#b8963e` — Mako® robotics

## Non-Physician Trial Sites
| Site | Domain |
|---|---|
| Carl Week Neurofeedback | carlweekneurofeedback.worker-bee.app |
| Trevor Smith Peds | trevorsmithpeds.worker-bee.app |
| Swig Drinks | swigdrinks.worker-bee.app |
| Firehouse Subs TF | firehousesubstf.worker-bee.app |

## Iterations Log
`/Users/drive/build-iterations/iterations-log.json`

## Superseded By
[[work/active/wb-pipeline]] — 7-phase agent droid pipeline with observable handoffs
