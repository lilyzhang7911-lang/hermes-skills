---
name: coding-agent-provider-config
description: Custom provider config for OpenCode (DashScope, Ollama).
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [Coding-Agent, Provider-Config, DashScope, OpenAI-Compatible, Setup]
    related_skills: [opencode, claude-code, codex]
---

# Coding Agent Provider Configuration

Configure custom AI providers for coding agents (OpenCode, Claude Code, Codex) when built-in providers don't cover your needs — especially domestic Chinese providers (DashScope, SiliconFlow) or local models (Ollama).

## When to Use

- Setting up OpenCode with a non-built-in provider (DashScope, SiliconFlow, Ollama)
- Configuring domestic Chinese providers to avoid proxy requirements
- Connecting local models (Ollama, llama.cpp) to coding agents
- Any OpenAI-compatible API endpoint needs to work with autonomous coding tools

## Core Concept

Most coding agents work with any OpenAI-compatible `/v1/chat/completions` endpoint. The key is:

1. **Provider config** tells the agent where to send requests
2. **Model config** tells it what models are available and their limits
3. **API key** authenticates the requests (via environment variable)

## OpenCode Custom Provider Setup

OpenCode uses `~/.config/opencode/opencode.jsonc` for provider configuration.

### Minimal Config Template

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "<provider-id>": {
      "npm": "@ai-sdk/openai-compatible",     // CRITICAL: must be this exact value
      "name": "Display Name",
      "options": {
        "baseURL": "https://api.example.com/v1",
        "apiKey": "{env:MY_API_KEY}"        // reads from env var
      },
      "models": {
        "model-id": {
          "name": "Display Name",
          "limit": { "context": 131072, "output": 16384 }
        }
      }
    }
  },
  "model": "<provider-id>/<model-id>",
  "small_model": "<provider-id>/<cheap-model>"
}
```

### Critical Requirements

1. **`npm` MUST be `"@ai-sdk/openai-compatible"`** — using `@ai-sdk/openai` or other values causes silent failures
2. **`{env:VAR_NAME}`** reads API key from environment — never hardcode keys in config
3. **Model ID** (the key in `models` object) is what gets sent to the API; `name` is just UI display
4. After config change, verify: `opencode models <provider-id>`

## Common Provider Endpoints

| Provider | baseURL | Auth Env Var |
|----------|---------|--------------|
| 阿里云百炼 (DashScope 按量) | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `DASHSCOPE_API_KEY` |
| 阿里云百炼 Token Plan | `https://token-plan.cn-beijing.maas.aliyuncs.com/compatible-mode/v1` | `DASHSCOPE_API_KEY` |
| 阿里云百炼 Coding Plan | `https://coding-plan.cn-beijing.maas.aliyuncs.com/compatible-mode/v1` | `DASHSCOPE_API_KEY` |
| 硅基流动 (SiliconFlow) | `https://api.siliconflow.cn/v1` | `SILICONFLOW_API_KEY` |
| 本地 Ollama | `http://localhost:11434/v1` | (none needed) |
| OpenRouter | `https://openrouter.ai/api/v1` | `OPENROUTER_API_KEY` |

**CRITICAL**: 阿里云百炼有三个独立系统（按量计费、Token Plan、Coding Plan），API Key 和 Base URL **互不相通，不可混用**。Token Plan 的 Key 格式为 `sk-sp-` 开头且包含 `.` 分隔符（JWT 格式），与标准 `sk-` 格式完全不同。详见 `references/chinese-cloud-providers.md`。
| 本地 Ollama | `http://localhost:11434/v1` | (none needed) |
| OpenRouter | `https://openrouter.ai/api/v1` | `OPENROUTER_API_KEY` |

**CRITICAL**: 阿里云百炼有三个独立系统（按量计费、Token Plan、Coding Plan），API Key 和 Base URL **互不相通，不可混用**。Token Plan 的 Key 格式为 `sk-sp-` 开头且包含 `.` 分隔符（JWT 格式），与标准 `sk-` 格式完全不同。详见 `references/chinese-cloud-providers.md`。
| 本地 Ollama | `http://localhost:11434/v1` | (none needed) | — |
| OpenRouter | `https://openrouter.ai/api/v1` | `OPENROUTER_API_KEY` | `sk-or-xxxx` |

**CRITICAL:** Token Plan / Coding Plan / 按量计费 的 API Key 和 Base URL **互不相通，不可混用**。Token Plan 的 Key 以 `sk-sp-` 开头且包含 `.` 分隔符（JWT 格式），与标准 `sk-` 格式完全不同。详见 `references/chinese-cloud-providers.md`。

## DashScope (阿里云百炼) Example

Full working config for Qwen models:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "dashscope": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "阿里云百炼 (DashScope)",
      "options": {
        "baseURL": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "apiKey": "{env:DASHSCOPE_API_KEY}"
      },
      "models": {
        "qwen3-coder-plus": {
          "name": "Qwen3-Coder Plus (编码最强)",
          "limit": { "context": 262144, "output": 65536 }
        },
        "qwen3-235b-a22b-thinking-2507": {
          "name": "Qwen3-235B Thinking (深度推理)",
          "limit": { "context": 262144, "output": 65536 }
        },
        "qwen-plus": {
          "name": "Qwen Plus (通用高性价比)",
          "limit": { "context": 131072, "output": 16384 }
        }
      }
    }
  },
  "model": "dashscope/qwen3-coder-plus",
  "small_model": "dashscope/qwen-plus"
}
```

### Environment Setup

```bash
export DASHSCOPE_API_KEY="***"
# Persist in shell config:
echo 'export DASHSCOPE_API_KEY="***"' >> ~/.zshrc
```

### Verification

```bash
opencode models dashscope
opencode run "Respond with exactly: SMOKE_TEST_OK"
```

## Pitfalls

1. **`npm` field is critical** — wrong value causes silent failures with no clear error message
2. **API key must be in env before running** — `{env:VAR_NAME}` is evaluated at runtime, not config-load time
3. **Context limits matter** — set realistic `limit.context` and `limit.output`; OpenCode uses these for context management and will truncate if limits are wrong
4. **Model ID vs Display Name** — the key in `models` is the actual API model ID; `name` is just for UI
5. **Domestic providers are faster in China** — DashScope/SiliconFlow don't need proxy, unlike OpenRouter/OpenAI
6. **`small_model` saves money** — set it to a cheap model for background tasks; main `model` handles primary coding work
7. **Hermes auto-redacts API keys** — when you paste an API key in chat, Hermes replaces it with `***` in all tool calls. This means **you cannot set API keys via Hermes tools** (terminal, execute_code, etc.). User must set them manually in their own terminal:
   ```bash
   export DASHSCOPE_API_KEY="***"
   echo 'export DASHSCOPE_API_KEY="***"' >> ~/.zshrc
   ```
8. **Alibaba Cloud Bailian has THREE variants** — 按量计费 (standard), Token Plan, and Coding Plan. They use **different endpoints and incompatible API keys**. Token Plan keys have format `sk-sp-x.xxx.x...` (JWT-like with dots), NOT the standard `sk-xxxx` format. Check `references/chinese-cloud-providers.md` for details.

## Procedure

1. Identify provider endpoint and auth method
2. Write config to `~/.config/opencode/opencode.jsonc` (or project-level `./opencode.json`)
3. Set API key in environment variable
4. Verify with `opencode models <provider-id>`
5. Smoke test with `opencode run "simple prompt"`
6. If fails, check:
   - `npm` field is `"@ai-sdk/openai-compatible"`
   - `baseURL` ends with `/v1` (or correct path for provider)
   - API key env var is set: `echo $MY_API_KEY`
   - Provider endpoint is reachable: `curl -I <baseURL>`
