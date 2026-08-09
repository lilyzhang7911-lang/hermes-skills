---
name: llm-provider-router
description: "Set up and configure LLM aggregation routers (e.g. FreeLLMAPI, Sub2API) with multiple upstream provider keys for unified OpenAI-compatible API access."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [macos, linux]
tags: [llm, api-router, free-llm-api, multi-provider, key-management]
---

# LLM Provider Router Setup

## Use Case

When you need to aggregate multiple AI model providers behind a single OpenAI-compatible API endpoint — for cost savings (free tiers), unified access, or routing flexibility.

## Core Workflow

### 1. Deploy the Router

```bash
cd freellmapi-main
npm run build && node server/dist/index.js &
# Or: npm run dev (background mode)
```

Default port: **3001** — verify with `curl http://localhost:3001/api/health`

### 2. Create Dashboard Admin Account

FreeLLMAPI requires a dashboard account before adding keys:

```bash
curl -s -X POST http://localhost:3001/api/auth/setup \
  -H "Content-Type: application/json" \
  -d '{"email":"<your-email>","password":"<strong-password>"}'
# Returns: { token, email } — save the token!
```

### 3. Add Provider Keys via API

Use the Dashboard token to add keys programmatically:

```bash
DASH_TOKEN="<your-dashboard-token>"

curl -s -X POST http://localhost:3001/api/keys \
  -H "Authorization: Bearer $DASH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"platform":"<provider-name>","key":"<api-key>","label":"<human-label>"}'
```

Supported platforms (from `server/src/providers/index.ts`):
- `groq`, `openrouter`, `github`, `mistral`, `cerebras`, `nvidia`, `huggingface`, `zhipu`, `ollama`, `kilo`, `pollinations`, `ovh`, `llm7`, `opencode`, `agnes`, `reka`, `siliconflow`, `routeway`, `bazaarlink`, `ainative`, `aion`, `requesty`, `navy`, `nara`, `custom`

### 4. Verify with Auto-Routing Test

```bash
curl -s http://localhost:3001/v1/chat/completions \
  -H "Authorization: Bearer <your-router-api-key>" \
  -H "Content-Type: application/json" \
  -d '{"model":"auto","messages":[{"role":"user","content":"Say hi in one word."}]}' | python3 -m json.tool
```

If `status` shows `"unknown"` on health check, wait ~5 minutes for auto-detection or test directly via `/v1/chat/completions`.

## Provider Key Acquisition Guide

### 🔹 No phone number / no credit card required (A-class)

| Provider | Where to get key | Notes |
|----------|-----------------|-------|
| **Groq** | console.groq.com/keys — email signup | Llama 3.3/4, GPT-OSS, Qwen3 |
| **OpenRouter** | openrouter.ai/keys — email signup | 21+ free models available |
| **GitHub Models** | GitHub PAT (settings/tokens) | GPT-4.1, GPT-4o |
| **Mistral** | console.mistral.ai/api-keys — email signup | Large 3, Medium 3.5, Codestral |
| **Cohere** | cohere.com/dashboard — email signup | Command R+, Command-A |
| **Cloudflare** | Cloudflare dashboard → Manage account → API Tokens | Kimi K2, GLM-4.7, GPT-OSS |
| **NVIDIA NIM** | build.nvidia.com — Google/Microsoft login | 40 RPM free (evaluation) |
| **HuggingFace** | huggingface.co/settings/tokens — email signup | DeepSeek V4, Kimi K2.6, Qwen3 |
| **SiliconFlow** | siliconflow.com — email signup | Chat/Image/Audio routing |
| **Reka** | platform.reka.ai — email signup | Flash, Edge |

### 🔹 May require phone number or additional verification (B-class)

| Provider | Notes |
|----------|-------|
| **Google Gemini** | aistudio.google.com/apikey — may need phone verification |
| **Z.ai (智谱)** | docs.z.ai — confirm if Chinese phone needed |
| **Ollama Cloud** | ollama.com — email signup, no credit card |
| **OpenCode Zen** | opencode.ai/auth — free account, pay-as-you-go |

### 🔹 Anonymous/Keyless providers (no key needed)

- **Kilo Gateway** — `key: ""` in API call
- **Pollinations** — `key: ""`
- **OVH AI Endpoints** — `key: ""`
- **AI Horde** — `key: ""`

## Retrieving Existing Keys from Hermes Config

If a key was already configured in Hermes Agent, check these locations:

```bash
# 1. Check .env files (may contain masked values)
grep "OPENROUTER_API_KEY" ~/.hermes/.env | cut -d'=' -f2-

# 2. Check config.yaml for provider blocks
grep -A5 "openrouter:" ~/.hermes/config.yaml

# 3. Check environment variables
env | grep -i "openrouter\|OPENROUTER"

# 4. macOS Keychain (if stored there)
security find-generic-password -l "openrouter" 2>/dev/null
```

⚠️ **Note**: `.env` files may show masked values (`sk-or-...7273`). For the full key, go to the provider's dashboard directly.

## Debugging Workflow

When `/v1/chat/completions` returns 400 or timeout:

1. **Verify API key first**: `curl http://localhost:3001/v1/models -H "Authorization: Bearer <key>"` — if this succeeds, auth is fine
2. **Test with specific model name** (not `"auto"`): Use a concrete model like `kimi-k2.6`, `gpt-4o-mini`, etc. to isolate routing vs auth issues
3. **Check upstream provider keys**: Verify each configured key works at the provider's native API first
4. **Review FreeLLMAPI logs**: Run `node server/dist/index.js` in foreground or check background process output for upstream call failures

## Debugging Workflow

When `/v1/chat/completions` returns 400 or timeout:

1. **Verify API key first**: `curl http://localhost:3001/v1/models -H "Authorization: Bearer <key>"` — if this succeeds, auth is fine
2. **Test with specific model name** (not `"auto"`): Use a concrete model like `kimi-k2.6`, `gpt-4o-mini`, etc. to isolate routing vs auth issues
3. **Check upstream provider keys**: Verify each configured key works at the provider's native API first — dashboard shows "configured" but keys may be expired/invalid
4. **Review FreeLLMAPI logs**: Run `node server/dist/index.js` in foreground or check background process output for upstream call failures

## Auth Architecture (Critical)

FreeLLMAPI has TWO layers of authentication:

### Layer 1: Provider Keys (in SQLite DB)
- Stored encrypted in `api_keys` table using AES-GCM with ENCRYPTION_KEY from `.env`
- Schema: `id, platform, label, encrypted_key, iv, auth_tag, status, enabled, created_at, last_checked_at, base_url`
- NOT in `.env` — only `ENCRYPTION_KEY` is in `.env`

### Layer 2: Unified API Key (in DB settings table)
- The `/v1/models` and proxy endpoints require a "unified API key" sent as `Authorization: Bearer <key>` header
- This is the router-level auth key, stored in the `settings` table of SQLite
- Retrieved via: `sqlite3 server/data/freeapi.db "SELECT value FROM settings WHERE key='unified_api_key';"`

### Common Confusion
When `/v1/models` returns "Invalid API key":
- **Wrong**: Sending a provider key (e.g., Groq key) as Bearer token
- **Right**: Sending the unified API key from the settings table

## Database Inspection Commands

```bash
# Check configured providers and their status
sqlite3 server/data/freeapi.db "SELECT id, platform, label, status, enabled FROM api_keys;"

# Get the unified API key (needed for /v1/models auth)
sqlite3 server/data/freeapi.db "SELECT value FROM settings WHERE key='unified_api_key';"

# Check all settings
sqlite3 server/data/freeapi.db "SELECT * FROM settings;"
```

## Pitfalls
## Pitfalls

| Symptom | Cause | Fix |
|---------|-------|-----|
| **tsx watch hangs silently, /v1/chat/completions times out** | `tsx watch src/index.ts` (dev/watch mode) can get stuck on upstream calls; the dist build (`node server/dist/index.js`) is reliable. If you started with `npm run dev`, kill it and restart with `node server/dist/index.js &` | Kill tsx process, start dist version: `pkill -9 -f "tsx.*src/index"; node server/dist/index.js &` |
| Health check shows `status: unknown` | Auto-detection runs every 5 minutes | Wait or test via `/v1/chat/completions` directly |
| "/v1/models" succeeds but "/v1/chat/completions" times out | "auto" router stuck selecting upstream; auth is fine | Test with a specific model name (e.g. `kimi-k2.6`) instead of `"auto"` to isolate routing vs auth issues |
| Hermes config set provider but still 400 in new conversation | Hermes caches old session; needs restart or new chat | Start a fresh conversation after changing provider config |
| SQLite query fails with "no such column: name" on settings table | FreeLLMAPI DB uses `key`/`value` columns, not `name`/`value` | Use `SELECT * FROM settings WHERE key='unified_api_key'` |

## Quick Verification Script

After starting FreeLLMAPI, run this to verify everything works:

```bash
#!/bin/bash
# Quick verification of FreeLLMAPI setup
echo "=== FreeLLMAPI Status Check ==="
curl -s --max-time 5 http://localhost:3001/v1/models | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'✅ Models available: {len(d.get(\"data\",[]))}')" 2>/dev/null || echo "❌ Server not running"

echo ""
echo "=== Chat Completion Test ==="
curl -s --max-time 30 http://localhost:3001/v1/chat/completions \
  -H "Authorization: Bearer freellmapi-abc0d42aa26af2477b90b2e37b7a139d9fc7c1fdecef02af" \
  -H "Content-Type: application/json" \
  -d '{"model":"auto","messages":[{"role":"user","content":"Say hi in one word."}]}' | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'✅ Response: {d[\"choices\"][0][\"message\"][\"content\"]}')" 2>/dev/null || echo "❌ Chat completion failed"
```

## Integration with Hermes Agent

Once FreeLLMAPI is running on port 3001, configure it as a custom OpenAI-compatible provider in `~/.hermes/config.yaml`:

```yaml
providers:
  freellmapi:
    base_url: http://localhost:3001/v1
    api_key: freellmapi-abc0d42aa26af2477b90b2e37b7a139d9fc7c1fdecef02af
```

Then use any model via `freellmapi/<model-id>` or `auto` for smart routing.

## Related Skills

- `hermes-multi-model-workflows`: Configure multi-model workflows in Hermes (main session + delegation).
- `sub2api-analysis`: Analyze Sub2API project architecture and deployment.