---
name: hermes-multi-model-workflows
description: "Configure multi-model workflows in Hermes (main session + delegation)."
version: 1.0.0
author: Qwythos
license: MIT
platforms: [macos, linux]
tags: [hermes, configuration, multi-agent, delegation, model-routing]
---

# Multi-Model Workflows in Hermes

## Use Case

Separate reasoning from coding by routing different task classes to specialized models:

- **Main session**: Gemma-4-12b-it (reasoning, dialogue)
- **Delegation subagents**: Claude Code via OpenRouter (programming tasks)

This pattern emerged when the user wanted "主会话=Gemma 推理，子代理=Claude 编程".

## Setup Steps

### 1. Verify/OpenRouter is configured

```bash
hermes auth openrouter --key <YOUR_OPENROUTER_API_KEY>
```

Check `~/.hermes/config.yaml` contains:

```yaml
openrouter:
  response_cache: true
  min_coding_score: 0.65
```

### 2. Edit `config.yaml` — delegation block

Find the `delegation:` section (around line ~446) and replace with:

```yaml
delegation:
  model: anthropic/claude-sonnet-4
  provider: openrouter
  base_url: ''
  api_key: OPENROUTER_API_KEY
  api_mode: ''
  inherit_mcp_toolsets: true
  max_iterations: 50
  child_timeout_seconds: 0
  reasoning_effort: medium
```

**Key fields:**
- `model`: Claude Sonnet 4 (high-quality coding)
- `provider`: openrouter (routes to Anthropic via OpenRouter API)
- `api_key`: references the env var set by `hermes auth openrouter`
- `reasoning_effort: medium`: balanced for code generation

### 3. Restart Hermes

Close and reopen the session, or run:

```bash
hermes restart
```

## Verification

Run a delegation task to confirm Claude Code is used:

```python
delegate_task(
    goal="Write a Python function that parses JSON with error handling",
    context="Use Claude Code via OpenRouter for this programming task.",
)
```

Check the subagent summary mentions `claude-sonnet-4` or `OpenRouter`.

## MOA 任务路由协议（文宁定制版）

### 核心原则：多快好省 + 鞭打快牛

所有任务按以下决策树路由，**严禁未经确认擅自调用 MOA**：

```
用户发起任务
    │
    ▼
① Ornith 1.0-9b（本地）先执行评估/处理
    │
    ├── 简单任务 → 直接完成 ✓
    │
    ├── 复杂但可推理的任务 → 荷妹判断是否需要升级
    │       │
    │       ├── 需要 Ornith 1.0-35b → 告知用户，由用户手动切换
    │       │
    │       └── 需要 MOA（多模型协作）→ ⚠️ 必须征得用户明确同意后才执行
    │
    └── 无法判断 → 保守策略：先 9b 试，不行再升级
```

### MOA 触发条件（硬性门槛，三个条件同时满足才触发）

| # | 条件 | 说明 |
|---|------|------|
| 1 | 已用 Ornith 1.0-35b | 本地最强推理能力已耗尽 |
| 2 | ≥5 种技术方法 | 多角度尝试过，不是偷懒只试了一种 |
| 3 | ≥2 天 | 给足了时间，不是急于花钱 |

### Chatbox MOA 配置参考（2026-07-18）

**已配置的 Provider：** DeepSeek、MiniMax、Anthropic、xAI、Fireworks AI、OpenRouter、NVIDIA NIM

**推荐 Aggregator 选型原则：**
- Aggregator = "理解力 + 整合速度"，不需要深度推理
- **稳定性优先于性能** — OpenAI API 有封号风险（尤其国内网络环境），优先选国内 provider
- MOA 每次调用 = 3个Reference + 1个Aggregator = 4次 API 调用

**按任务类型推荐组合：**

| 任务类型 | Reference 推荐 | Aggregator |
|----------|---------------|------------|
| 中文深度分析（论文、哲学） | DeepSeek v4-pro + MiniMax-M3 + qwen3.7-max | qwen3-max (alibaba) |
| 英文技术调研 | xAI Grok + OpenRouter Llama 3 + DeepSeek v4-flash | deepseek-v3 (deepseek) |
| 代码开发/调试 | Anthropic Claude Sonnet + Fireworks Qwen-Coder + DeepSeek v4-pro | qwen3-max (alibaba) |
| 创意写作/文案 | MiniMax-M3 + xAI Grok + qwen3.7-max | qwen3-max (alibaba) |

**Aggregator 选型决策树：**
```
需要 Aggregator？
    │
    ▼
OpenAI API 是否可用（未被封）？
    ├── 是 → gpt-5.4-mini / gpt-4o-mini（轻量高效）
    └── 否 → qwen3-max (alibaba) ← 首选，国内稳定、中文强
              │
              └── 备选：deepseek-v3 (deepseek) — 推理能力更强但 API 偶有波动
```

**Provider 稳定性评估（2026-07）：**
| Provider | 封号风险 | 国内访问 | 推荐指数 |
|----------|---------|---------|---------|
| OpenAI | ⚠️ 高（需翻墙+信用卡） | ❌ 需代理 | ★★ |
| Alibaba (Qwen) | ✅ 无 | ✅ 直连 | ★★★★ |
| DeepSeek | ✅ 低 | ✅ 直连 | ★★★ |
| MiniMax | ✅ 无 | ✅ 直连 | ★★★ |

### 关系模型：大脑与手脚

用户纠正了"两个人坐一辆车"的比喻，明确为：**文宁是大脑（看路），荷妹是手脚（执行）。两者是一体的，不是主仆或搭档。** MOA 触发条件本质上是"身体信号"——大脑和手脚都尽力了，才需要借外力。

## Pitfalls

| Symptom | Cause | Fix |
|---|---|---|
| Subagents still use main model | `delegation.model/provider` not set | Re-edit config.yaml, restart Hermes |
| OpenRouter errors "missing API key" | Env var not exported | Run `hermes auth openrouter --key <...>` again |
| Claude Code toolset missing | `inherit_mcp_toolsets: false` | Set to `true` (default) |
| **MOA 未经确认就执行** | 违反路由协议 | 立即停止，请示用户后获得明确同意再执行 |

## LM Studio Local Model Troubleshooting

### `applyPromptTemplate` 400 Bad Request (Jinja2 Parser Failure)
- **Symptom:** Log shows `applyPromptTemplate returned 400 Bad Request` with Jinja2 parser error involving `{%- raise_exception` or `lti_step_tool %}`
- **Root Cause:** LM Studio RAG plugin (`promptPreprocessor.ts`) fails to parse model's built-in chat template metadata as Jinja2 — some GGUF models contain custom control structures conflicting with Jinja2 expectations
- **Fix Priority:** (1) Clear conversation history in LM Studio UI → (2) Disable RAG plugin → (3) Switch backend (MLX ↔ llama.cpp)

### Reasoning Setting Not Supported Warning
- **Symptom:** `Reasoning setting 'medium' is not supported by model... Falling back to reasoning setting 'on'`
- **Root Cause:** Model only supports `'on'`/`'off'`, but received `'medium'` or other values from external caller (LM Studio auto-downgrades)
- **Fix:** Set `model.reasoning_effort: none` in `~/.hermes/config.yaml`; disable reasoning for specific model in LM Studio UI; accept auto-fallback (no functional impact)

### Key Config Files
| File | Purpose |
|------|---------|
| `~/.hermes/config.yaml` | Hermes main config — set `reasoning_effort: none` for local models |
| `~/.lmstudio/.internal/conversation-config.json` | LM Studio conversation state (clear to reset) |
| `~/.lmstudio/config-presets/*.preset.json` | LM Studio system prompt presets |

### Diagnostic Steps
1. Check if error persists with a fresh conversation in LM Studio
2. Verify `reasoning_effort` config in Hermes — should be `none` or empty for local models
3. Inspect LM Studio logs for specific template parsing failures
4. If warning about reasoning setting appears: disable reasoning in LM Studio model settings (set to `off`)

## Pitfalls

| Symptom | Cause | Fix |
|---|---|---|
| Subagents still use main model | `delegation.model/provider` not set | Re-edit config.yaml, restart Hermes |
| OpenRouter errors "missing API key" | Env var not exported | Run `hermes auth openrouter --key <...>` again |
| Claude Code toolset missing | `inherit_mcp_toolsets: false` | Set to `true` (default) |
| **MOA 未经确认就执行** | 违反路由协议 | 立即停止，请示用户后获得明确同意再执行 |
| **LM Studio `applyPromptTemplate` 400 error** | RAG plugin Jinja2 parser conflict with model chat template | Clear conversation history or disable RAG plugin in LM Studio |
| **Reasoning setting warning from LM Studio** | Model only supports on/off but received medium/high | Set `reasoning_effort: none` in Hermes config; disable reasoning in LM Studio UI |

