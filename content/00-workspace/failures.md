# Portfolio Failure Log

Bugs, gotchas, and pitfalls that have burned us in production. Each entry exists so the same class of mistake never repeats.

## How to Use This File

- **Adding an entry**: every time something breaks in prod or wastes >30 min in dev, add a row
- **Format**: keep it factual. What happened, what to do instead, when it happened, where
- **Reading**: before starting work on a project, skim entries tagged with that project
- **Pruning**: don't prune. Old entries stay as institutional memory. If a class of failure is no longer possible, mark it `[resolved]` but leave the record

## Format

```
- YYYY-MM-DD [project] What happened — How to avoid — Where to learn more
```

## Active Entries (seeded from memory and CLAUDE.md context)

### Auth and Supabase

- 2026-05 [jrs, silvercreek] Mixing `/admin` cookie auth with `/portal` Supabase JWT causes silent session overwrites in prod — keep them fully separate, see ADR 0006
- ~2026-04 [multiple] Setting `@supabase/ssr` to "latest" broke SSR cookie handling at 2.50.x — pin to a known-good minor version
- 2026-05 [multiple] Importing `lib/supabase/admin.ts` from a Client Component leaks service-role key into the browser bundle — only import from server contexts
- 2026-05 [multiple] Calling `cookies()` inside a Client Component returns undefined — only `lib/supabase/server.ts` reads cookies; refactor to pass values via props or Route Handler

### Deployment and Cron

- 2026-05 [silvercreek] CRON_SECRET mismatch between Vercel cron and the Cloudflare Worker cron caused silent dispatch failures — secret must match in BOTH platforms
- 2026-05 [multiple] `next dev` without `-H 0.0.0.0` works on localhost but breaks Tailscale preview at 100.117.143.57 — always pass the flag
- 2026-05 [climb-*] Forgetting to set `nodejs_compat` in wrangler.jsonc → cryptic runtime errors about missing Node APIs
- 2026-05 [language-lens-elite] Using `fs.readFile` at runtime works in `npm run dev` (Node) but fails in Cloudflare Workers prod — bundle the file or use ASSETS binding

### Deployment Platform Selection

- 2026-05-22 [climb-france] Wrangler deploy requires interactive `wrangler login` — when the token expires the build completes but the deploy step hits an auth wall with no fallback. Total cost: ~45 min of build time + node_modules corruption diagnosis + wasted context. The correct response was to switch platforms immediately. Lesson: marketing/affiliate sites go on Vercel per ADR-0003. Cloudflare Workers is for `worker-bee.app` subdomain projects with D1/R2/KV bindings. The moment wrangler auth fails, apply the 5-method rule: (1) `vercel --prod` — CLI auth lives in `~/.config/vercel/auth.json`, survives session restarts; (2) Vercel GitHub auto-deploy — if the repo is connected, every push to main deploys without any command; (3) `CLOUDFLARE_API_TOKEN` in env — check `.env` first, the token may already be present under a slightly different name; (4) `mcp__claude_ai_Vercel__deploy_to_vercel` — Vercel MCP authenticated via Claude.ai, no CLI needed; (5) GitHub Actions — one-time setup, then every push deploys autonomously forever. Never burn more than 5 minutes on a dead auth path before switching options.
- 2026-05-22 [global] `CLOUDFLARE_TOKEN` vs `CLOUDFLARE_API_TOKEN` — wrangler reads `CLOUDFLARE_API_TOKEN` specifically. The system `.env` had the token stored as `CLOUDFLARE_TOKEN`. One-character name mismatch caused a silent auth failure when the correct token was already present. Always check env before assuming the token is missing: `env | grep -i cloudflare` or `cat ~/.env | grep -i cloud`.
- 2026-05-22 [global] `npm install --force` does NOT fix incomplete node_modules extractions — npm checks directory existence, not file completeness. If a package directory exists but is missing files, `npm install` reports "up to date" and does nothing. Fix: `npm ci` (deletes and reinstalls from lockfile) or copy the complete package from a sibling project with matching dependencies.

### Build and Tooling

- 2026-05 [devtools] node-pty on macOS arm64 throws `posix_spawnp failed` on first run — run `npm rebuild node-pty && xattr -cr node_modules/node-pty/prebuilds/darwin-arm64/ && chmod +x .../spawn-helper`
- 2026-05 [global] Calling `claude-flow` as `npx @claude-flow/cli@latest` re-downloads on every run — use the global `claude-flow` binary
- 2026-05 [global] Editing `.env.local` and expecting Next.js to hot-reload env vars — it does not, restart `npm run dev`

### UI and Verification

- 2026-05 [salvorias, others] Using `screenshot.js` to verify scroll/hover/animation produces false-positive quality scores — screenshots freeze mid-animation, always use `record.js` for any motion-driven UI
- 2026-05 [language-lens-elite] Adding a new tab without updating both `TabKey` union AND `TAB_COMPONENTS` map — TypeScript doesn't catch the missed map entry, runtime fails on tab switch
- 2026-05 [climb-*] Filling route `images` blocks from a small pool of Unsplash IDs caused the same crux photo to appear on 6/8 routes and the same approach on 5/8 — looks cheap, kills credibility. Assign unique images per slot per site. `grep -oE 'photo-[A-Za-z0-9]+|/images/routes/[^'"'"']+' routes.ts | sort | uniq -d` catches duplicates before they ship. See ADR-0013

### Storage and Content

- 2026-05 [jrs] Creating markdown files for blog posts — blog reads from `lib/articles.ts` only, the markdown is silently ignored. See ADR 0004
- 2026-05 [manage-worker-bee] Editing a blueprint without migrating legacy `{nodes, edges}` flat format → data loss on save. See `lib/blueprintStore.ts` migration path

### AI / LLM

- 2026-05 [language-lens-elite] Sending unvalidated user input to Anthropic API — prompt injection risk. Always Zod-validate before LLM call
- 2026-05 [multiple] Burning Sonnet/Opus on mechanical tasks (file rename, image resize, git commits) — route to Haiku via Agent tool with `model: "haiku"`. See ADR 0005

### Performance

- 2026-05 [marketing sites] LCP > 2.5s because hero image wasn't `priority` — biggest preventable SEO loss. Always set `priority` + `fetchPriority="high"` + `sizes` on above-the-fold images
- 2026-05 [marketing sites] Missing `sizes` prop on `next/image` → wrong responsive variant served, CLS spikes

### CLAUDE.md and Process

- 2026-05 [global] Editing CLAUDE.md frequently busts the cache on every session — treat it like an API contract, edit infrequently
- 2026-05 [global] Writing 11 separate CLAUDE.md files independently — 80% duplication, drift between projects. Solved by ADR 0007 (layered model)

## Templates for Common Entry Patterns

When you add an entry, pick the format that matches:

```
- YYYY-MM-DD [project] [Setting X to Y] caused [outcome] — [fix or how to avoid]
- YYYY-MM-DD [project] [Action that looked safe] silently [bad result] — [check N before doing it]
- YYYY-MM-DD [project] [Tool / library] requires [non-obvious setup step] — [exact command]
- YYYY-MM-DD [project] [Pattern] worked in dev but failed in prod because [reason] — [how to test for it]
```

## Tags

Common tags so entries can be grouped:

`[auth]` `[deploy]` `[cron]` `[supabase]` `[cloudflare]` `[vercel]` `[performance]` `[seo]` `[ai]` `[ui]` `[build]` `[secrets]` `[dns]`

## Adding to This File

This is the cheapest, highest-leverage doc to maintain. Every entry pays for itself the first time it prevents a repeat. Don't curate — just append. A messy failure log is better than a clean one with no entries.
