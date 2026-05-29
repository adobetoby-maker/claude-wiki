---
ai-first: true
type: reference
date: 2026-05-28
tags: [credentials, secrets, api-keys, mcp, env-vars, security]
---

# Credential & Secret Finder

## For future Claude
When you need an API key, secret, or MCP token — look here first. This is the map of where every credential type lives on this machine and how to retrieve it. Never store actual values in the vault. Never ask Toby for a key until you've checked all locations below.

---

## Decision Tree — Where To Look First

```
Need a credential?
  ↓
1. Project .env.local → cat /Users/drive/<project>/.env.local
2. Vercel env (deployed values) → vercel env pull --cwd /Users/drive/<project>
3. manage.worker-bee.app/vault → browser UI, AES-256 encrypted store
4. macOS Keychain → security find-generic-password -s "<service>" -g
5. GitHub secrets → gh secret list --repo adobetoby-maker/<repo>
6. Cloudflare Worker secrets → wrangler secret list --cwd <path>
7. Hermes MCP tokens → ~/.hermes/profiles/local/mcp-tokens/
8. Claude plugin OAuth → ~/.claude/mcp-needs-auth-cache.json
```

---

## 1. Project .env Files

Most values are here. Always check this first.

```bash
cat /Users/drive/<project>/.env.local      # most common
cat /Users/drive/<project>/.env            # fallback
cat /Users/drive/.env                      # global fallback (top-level)
cat /Users/drive/.hermes/.env             # Hermes env
```

Known `.env.local` locations:
- `/Users/drive/mountain-edge-training/.env.local`
- `/Users/drive/mountain-edge-electrical/.env.local`
- `/Users/drive/mountain-edge-general/.env.local`
- `/Users/drive/marks-parent-coaching/.env.local`
- `/Users/drive/block-reign-site/.env.local`
- `/Users/drive/.claude-mem/.env`
- (run `find /Users/drive -maxdepth 2 -name ".env.local" 2>/dev/null | grep -v node_modules` to refresh)

---

## 2. Vercel Env Vars

Names visible via CLI. Values are encrypted — use `env pull` to get actual values locally.

```bash
# List names (shows environment: Production/Preview/Development)
vercel env ls --cwd /Users/drive/<project>

# Pull all values to .env.local (creates/overwrites the file)
vercel env pull --cwd /Users/drive/<project>

# Pull for specific environment
vercel env pull --environment production --cwd /Users/drive/<project>
```

Key variables by project:

| Project | Key Vars |
|---|---|
| language-lens-elite | ANTHROPIC_API_KEY, STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, SUPABASE_SERVICE_ROLE_KEY, SUPABASE_URL, VITE_SUPABASE_URL, VITE_SUPABASE_PUBLISHABLE_KEY, VITE_SUPABASE_PROJECT_ID, VITE_GA_ID |
| silver-creek-logistics | ADMIN_SECRET, CRON_SECRET, SUPABASE_SERVICE_ROLE_KEY, TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, TWILIO_FROM, TWILIO_DISPATCH_PHONES, QB_CLIENT_ID, QB_CLIENT_SECRET, QB_REDIRECT_URI, GMAIL_USER, GMAIL_APP_PASSWORD |
| jrs-auto-repair | NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY, ANTHROPIC_API_KEY |

---

## 3. manage.worker-bee.app Vault

Browser-based AES-256-GCM encrypted credential store. Source of truth for client credentials, admin passwords, and sensitive project notes.

```
URL: https://manage.worker-bee.app/vault
Auth: ADMIN_SECRET password (same as manage-worker-bee admin login)
Code: /Users/drive/manage-worker-bee/lib/vaultStore.ts
```

Stores: logins, API keys, database URLs, SSH keys, env files, notes.
Categories: `login` | `api-key` | `database` | `ssh` | `env` | `note`

Use this for: Anderton & Associates admin password, JRS admin password, any client-facing credentials.

---

## 4. macOS Keychain

Claude Code OAuth token and other system credentials.

```bash
# Claude Code OAuth token
security find-generic-password -s "Claude Code-credentials" -a "drive" -g 2>&1

# Search for any service
security find-generic-password -s "<service-name>" -g 2>&1

# List all keychain items (grep for service name)
security dump-keychain | grep "svce" | sort -u
```

Known keychain entries:
- `Claude Code-credentials` — Max OAuth token (account: drive)
- Check dump for Anthropic, Vercel, GitHub entries

---

## 5. GitHub Repository Secrets

Names visible, values NOT retrievable via CLI (set-only). If you need the value, check Vercel env or .env.local instead.

```bash
# List secrets for a repo
gh secret list --repo adobetoby-maker/<repo>

# Common repos with secrets:
gh secret list --repo adobetoby-maker/claude-wiki
# → VERCEL_TOKEN, VERCEL_ORG_ID, VERCEL_PROJECT_ID

gh secret list --repo adobetoby-maker/language-lens-elite
# → CLOUDFLARE_ACCOUNT_ID, CLOUDFLARE_API_TOKEN, VERCEL_TOKEN,
#   VERCEL_ORG_ID, VERCEL_PROJECT_ID,
#   VITE_SUPABASE_URL, VITE_SUPABASE_PUBLISHABLE_KEY, VITE_SUPABASE_PROJECT_ID
```

To set a new secret:
```bash
gh secret set SECRET_NAME --repo adobetoby-maker/<repo> --body "value"
# Or from file:
gh secret set SECRET_NAME --repo adobetoby-maker/<repo> < secret.txt
```

---

## 6. Cloudflare Worker Secrets

Names visible, values NOT retrievable. Must re-set if forgotten.

```bash
# List secrets for a Worker
wrangler secret list --cwd /Users/drive/<project>

# Set a secret
wrangler secret put SECRET_NAME --cwd /Users/drive/<project>

# Critical: CRON_SECRET must match between silver-creek Vercel AND CF Worker
# If they drift → dispatch silently rejects all requests
```

Cloudflare account credentials:
- Account ID: `53c642cb4b27b3b66628e60a7efa0c27` (in deploy-client.sh, workerbee-deploy.md)
- API token: baked into `~/deploy-client.sh` — do not echo; run the script directly

---

## 7. Hermes MCP OAuth Tokens

OAuth tokens for MCP servers used by Hermes `local` and `mini` profiles.

```
~/.hermes/profiles/local/mcp-tokens/
  ├── vercel.json            # Vercel OAuth token
  ├── vercel.client.json     # Vercel client config
  ├── vercel.meta.json       # token metadata
  ├── cloudflare-api.client.json
  ├── cloudflare-bindings.client.json
  ├── cloudflare-observability.client.json
  └── supabase.client.json

~/.hermes/profiles/mini/mcp-tokens/  (same structure)
```

Warning: These are OAuth tokens with scope `openid offline_access` — they work for MCP read/query operations but NOT for Anthropic API calls (different scope than `user:sessions:claude_code`).

To refresh an expired token: re-authenticate via the MCP server's OAuth flow in Claude Desktop or Hermes session.

---

## 8. Claude MCP Server Config

Two MCP servers configured directly in Claude Code settings:

```bash
cat ~/.claude/settings.json | python3 -c "
import sys, json
d = json.load(sys.stdin)
mcps = d.get('mcpServers', {})
for name, cfg in mcps.items():
    print(name, ':', cfg.get('command',''), cfg.get('url',''))
    for k,v in cfg.get('env',{}).items():
        print('  env:', k, '=', v[:20] + '...' if len(str(v)) > 20 else v)
"
```

Currently configured:
| Server | Type | Env key |
|---|---|---|
| `puppeteer` | node command | none |
| `comfyui` | npx command | `COMFYUI_URL`, `CIVITAI_API_TOKEN` |

Plugin MCP OAuth state (which plugins have completed OAuth):
```bash
cat ~/.claude/mcp-needs-auth-cache.json
# Shows: plugin:greptile, supabase, cloudflare-api, postman, etc.
```

Plugin credentials stored at: `~/.claude/.credentials.json`

---

## 9. Supabase Project Credentials

Supabase projects by ID — get keys from dashboard or `.env.local`:

| Project | Ref ID | Get keys at |
|---|---|---|
| manage-worker-bee | `qnrkifdbkcbacgznoabs` | supabase.com/dashboard/project/qnrkifdbkcbacgznoabs/settings/api |
| language-lens-elite | `pollhlkgltdkdskdzsgd` | supabase.com/dashboard/project/pollhlkgltdkdskdzsgd/settings/api |
| mountain-edge | `fddmjywlegzgwehsxzei` | supabase.com/dashboard/project/fddmjywlegzgwehsxzei/settings/api |

Keys from dashboard:
- `anon` / `publishable key` → safe in client-side code (`VITE_` or `NEXT_PUBLIC_` prefix)
- `service_role` key → server-side ONLY, bypasses RLS, never expose to browser

SQL editor for unapplied migrations: `supabase.com/dashboard/project/<ref>/sql/new`

---

## 10. Quick Retrieval Patterns

```bash
# "I need the Anthropic API key"
grep ANTHROPIC_API_KEY /Users/drive/language-lens-elite/.env.local 2>/dev/null ||
  vercel env pull --cwd /Users/drive/language-lens-elite && grep ANTHROPIC /Users/drive/language-lens-elite/.env.local

# "I need the Stripe secret key"
grep STRIPE_SECRET /Users/drive/language-lens-elite/.env.local 2>/dev/null ||
  vercel env pull --cwd /Users/drive/language-lens-elite

# "I need the Supabase service role key for project X"
grep SUPABASE_SERVICE_ROLE /Users/drive/<project>/.env.local 2>/dev/null

# "I need the CRON_SECRET for silver-creek"
grep CRON_SECRET /Users/drive/silver-creek-logistics/.env.local 2>/dev/null

# "I need the Cloudflare API token"
# → Check ~/deploy-client.sh (baked in) OR
# → Check ~/.hermes/profiles/local/mcp-tokens/cloudflare-api.client.json

# "I need the Vercel token for CI"
gh secret list --repo adobetoby-maker/<repo>  # confirms name
# Value → check project .env.local or manage.worker-bee.app/vault

# "I need the ComfyUI Civitai token"
cat ~/.claude/settings.json | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['mcpServers']['comfyui']['env']['CIVITAI_API_TOKEN'])"
```

---

## Never Do This

- Store actual secret values in vault MD files — vault is for navigation, not storage
- Ask Toby for a key before checking all 10 locations above
- Commit `.env.local` files to git — they're in `.gitignore` for a reason
- Use `grep -r` across all of `/Users/drive` looking for secrets — too broad, too slow
- Echo a secret into terminal output that appears in conversation history
