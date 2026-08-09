---
name: hindsight-deployment
description: "部署和配置 Hindsight AI 记忆系统（vectorize-io/hindsight），包括 Docker、pip/uv 安装和 LM Studio 本地模型集成。"
rational: true
version: 2.0.0
author: Hermes Agent + User
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [hindsight, deployment, ai-memory-system, docker, lmstudio]
---

# Hindsight Deployment

部署和配置 [Hindsight](https://github.com/vectorize-io/hindsight) — 一个开源 Agent 记忆系统（三层记忆模型：World Facts → Experience Facts → Mental Models），用于为 AI Agent 提供长期记忆能力。

## 触发条件

当用户需要安装、配置、启动或排查 Hindsight 记忆系统时使用此技能。

## 前置环境检查

在安装前，先确认环境状态（这些命令可并行执行）：

```bash
# 1. 架构确认（决定镜像大小和兼容性）
uname -m
# arm64 = Apple Silicon, x86_64 = Intel

# 2. Docker 是否可用
which docker && docker --version

# 3. LM Studio 是否运行（如使用本地模型）
curl -s --connect-timeout 3 http://localhost:1234/v1/models

# 4. uv 是否可用（Bare Metal 路径需要）
which uv
```

## 安装路径选择

| 路径 | 适用场景 | 镜像/包大小 | 复杂度 |
|------|----------|-------------|--------|
| **Docker（Full）** | 快速启动、开发测试 | ~3.7GB (ARM64) / ~9GB (AMD64) | 最低 |
| **Docker（Slim）** | 已有外部嵌入/重排服务 | ~500MB | 低 |
| **Bare Metal (pip)** | 独立服务、无 Docker | 取决于依赖 | 中 |
| **源码构建 (uv sync)** | 开发贡献、自定义修改 | 同上 | 高 |

### 路径 1：Docker 安装（推荐）

```bash
# 配置环境变量（根据实际 LLM 提供商修改）
export HINDSIGHT_API_LLM_PROVIDER=lmstudio          # 或 openai, anthropic, groq, ollama 等
export HINDSIGHT_API_LLM_MODEL=<your-model-name>      # LM Studio 中加载的模型名
export HINDSIGHT_API_LLM_API_KEY=not-needed           # LM Studio 不需要真实 key

docker run -d --name hindsight --restart unless-stopped \
  -p 8888:8888 -p 9999:9999 \
  -e HINDSIGHT_API_LLM_PROVIDER=$HINDSIGHT_API_LLM_PROVIDER \
  -e HINDSIGHT_API_LLM_MODEL=$HINDSIGHT_API_LLM_MODEL \
  -e HINDSIGHT_API_LLM_BASE_URL=http://host.docker.internal:1234/v1 \
  -e HINDSIGHT_API_LLM_API_KEY=$HINDSIGHT_API_LLM_API_KEY \
  -e HINDSIGHT_API_EMBEDDINGS_PROVIDER=local \
  -v hindsight-data:/home/hindsight/.pg0 \
  ghcr.io/vectorize-io/hindsight:latest
```

- **API 服务**: http://localhost:8888
- **控制面板 UI**: http://localhost:9999
- 嵌入式数据库 pg0 自动配置，数据持久化到 Docker volume `hindsight-data`

### 路径 2：Bare Metal (pip/uv) 安装

**推荐用 uv（Apple Silicon 上 pip3 可能不可用）**：

```bash
# 1. 创建虚拟环境
uv venv hindsight-venv --python 3.12

# 2. 安装（Full 版，开箱即用，约 209 个包）
uv pip install hindsight-api --python hindsight-venv/bin/python
# 慢网络时加长超时：UV_HTTP_TIMEOUT=300 uv pip install hindsight-api --python hindsight-venv/bin/python

# 3. 启动（嵌入式数据库 pg0）
export HINDSIGHT_API_LLM_PROVIDER=lmstudio
export HINDSIGHT_API_LLM_MODEL=<your-model-name>
export HINDSIGHT_API_LLM_API_KEY=not-needed
export HINDSIGHT_API_LLM_BASE_URL=http://localhost:1234/v1

hindsight-venv/bin/hindsight-api    # 启动 API（默认端口 8888）
# 数据库自动创建在 ~/.hindsight/data/
```

#### macOS LaunchAgent（开机自启 + 崩溃自动重启）

创建 `~/Library/LaunchAgents/com.vectorize.hindsight.plist`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.vectorize.hindsight</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/USER/hindsight-venv/bin/hindsight-api</string>
    </array>
    <key>EnvironmentVariables</key>
    <dict>
        <key>HINDSIGHT_API_LLM_PROVIDER</key>
        <string>lmstudio</string>
        <key>HINDSIGHT_API_LLM_MODEL</key>
        <string>YOUR-MODEL-NAME</string>
        <key>HINDSIGHT_API_LLM_API_KEY</key>
        <string>not-needed</string>
        <key>HINDSIGHT_API_LLM_BASE_URL</key>
        <string>http://localhost:1234/v1</string>
        <!-- 关键：清空 PYTHONPATH 防止 Hermes Agent 3.11 site-packages 污染 -->
        <key>PYTHONPATH</key>
        <string></string>
    </dict>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/USER/Library/Logs/Hindsight/hindsight.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/USER/Library/Logs/Hindsight/hindsight.err.log</string>
    <key>ThrottleInterval</key>
    <integer>10</integer>
</dict>
</plist>
```

加载和管理：
```bash
mkdir -p ~/Library/Logs/Hindsight
launchctl load ~/Library/LaunchAgents/com.vectorize.hindsight.plist   # 加载（立即启动）
launchctl unload ~/Library/LaunchAgents/com.vectorize.hindsight.plist # 卸载
launchctl list com.vectorize.hindsight                                 # 查看状态
# 更新 plist 后需 unload + reload
```

**关键配置项说明**：
- `KeepAlive: true` — 进程退出后自动重启（不要用 `SuccessfulExit: false`，kill 信号不会触发重启）
- `PYTHONPATH: ""` — 必须清空，否则 Hermes Agent 的 PYTHONPATH 环境变量会污染 Python 3.12 venv
- `ThrottleInterval: 10` — 重启间隔（秒），避免快速崩溃循环

### Hermes Agent MCP 集成

Hindsight API 内置 MCP Server（StreamableHTTP transport），端点在 `/mcp/`。添加到 Hermes：

```bash
# 添加 MCP server（交互式，回答 n, 空, Y）
hermes mcp add hindsight --url http://localhost:8888/mcp/
# 验证
hermes mcp test hindsight
hermes mcp list
```

在 Hermes `config.yaml` 中生成：
```yaml
mcp_servers:
  hindsight:
    url: http://localhost:8888/mcp/
    timeout: 120
    connect_timeout: 30
```

32 个 MCP 工具自动注册，前缀 `mcp_hindsight_*`（retain, recall, reflect, sync_retain 等）。新会话生效。

Slim 版：`uv pip install hindsight-api-slim --python hindsight-venv/bin/python`（需要外部嵌入和重排服务）

控制面板可选启动：
```bash
npx @vectorize-io/hindsight-control-plane --api-url http://localhost:8888
```

### 路径 3：源码构建（仅开发/贡献者）

```bash
git clone https://github.com/vectorize-io/hindsight.git
cd hindsight
uv sync --no-cache    # 约 258 个包
# 复制 .env.example 为 .env 并修改
bash scripts/dev/start-api.sh --port 8888
```

## 验证安装

```bash
# 健康检查
curl http://localhost:8888/health
# 预期: {"status":"healthy","database":"connected"}

# 1. 创建记忆库（PUT 方法，不是 POST）
curl -X PUT "http://localhost:8888/v1/default/banks/test-bank" \
  -H "Content-Type: application/json" \
  -d '{}'

# 2. 存储记忆（注意 items 数组包装，直接 content 会报 "Field required"）
curl -X POST "http://localhost:8888/v1/default/banks/test-bank/memories" \
  -H "Content-Type: application/json" \
  -d '{"items": [{"content": "Test memory from Hindsight deployment"}]}'

# 3. 检索记忆（POST + JSON body，不是 GET + query param）
curl -X POST "http://localhost:8888/v1/default/banks/test-bank/memories/recall" \
  -H "Content-Type: application/json" \
  -d '{"query": "test memory"}'
```

**API 路径结构**：所有端点都在 `/v1/default/banks/{bank_id}/...` 下。`default` 是命名空间，`bank_id` 是记忆库 ID。查看完整路由：`curl http://localhost:8888/openapi.json`

## 与 Hermes Agent 集成（MCP）

Hindsight API 内置 MCP Server（StreamableHTTP 传输），无需单独的可执行文件。MCP 端点就在 API 服务本身上。

### 关键发现

- `hindsight-local-mcp` 可执行文件**不是** MCP Server — 它只是用本地默认参数启动 `hindsight-api`。不要用它做 MCP 集成。
- MCP 端点 = `http://localhost:8888/mcp/`（API 服务启动后自动可用）
- 提供 32 个 MCP 工具：`retain`、`recall`、`reflect`、`sync_retain`、`create_bank`、`list_memories` 等

### 配置步骤

```bash
# 1. 确保 Hindsight API 正在运行（见上方启动命令）
curl http://localhost:8888/health

# 2. 添加 MCP server 到 Hermes Agent
hermes mcp add hindsight --url http://localhost:8888/mcp/

# 交互式提示流程：
#   Does this server require authentication? → n
#   API key / Bearer token → (直接回车)
#   Enable all 32 tools? → Y

# 3. 验证
hermes mcp list        # 确认 hindsight 出现且状态为 ✓ enabled
hermes mcp test hindsight  # 测试连接和工具发现
```

### 非交互式添加（用于脚本/cron）

`hermes mcp add` 三个提示可通过管道输入回答：

```bash
printf 'n\n\nY\n' | hermes mcp add hindsight --url http://localhost:8888/mcp/
```

### 启动后创建专用记忆库

```bash
curl -X PUT "http://localhost:8888/v1/default/banks/hermes-agent" \
  -H "Content-Type: application/json" \
  -d '{"name": "Hermes Agent Long-Term Memory", "mission": "Store cross-session knowledge for the Hermes Agent assistant."}'
```

### 生效条件

- MCP 工具在**会话启动时**加载，添加后需要**新会话**（`/reset` 或重启 Hermes）才能使用
- 新会话中工具名为 `mcp_hindsight_retain`、`mcp_hindsight_recall`、`mcp_hindsight_reflect` 等（`mcp_{server_name}_{tool_name}` 格式）
- Hermes 的环境变量过滤器不会传递 `PYTHONPATH` 给 stdio MCP 子进程，但 HTTP transport 不受影响

### 完整启动顺序（每次重启后）

```bash
# 1. 启动 LM Studio（加载模型）
# 2. 启动 Hindsight API（注意清除 PYTHONPATH）
export HINDSIGHT_API_LLM_PROVIDER=lmstudio
export HINDSIGHT_API_LLM_MODEL=ornith-1.0-35b-mtp-apex
export HINDSIGHT_API_LLM_API_KEY=not-needed
export HINDSIGHT_API_LLM_BASE_URL=http://localhost:1234/v1
env -u PYTHONPATH ~/hindsight-venv/bin/hindsight-api &

# 3. 等待首次启动（下载模型可能需 2-3 分钟）
sleep 90 && curl http://localhost:8888/health

# 4. 启动/重启 Hermes Agent（自动连接 Hindsight MCP）
```

## 关键配置参考

### LLM 提供商

| 提供商 | `HINDSIGHT_API_LLM_PROVIDER` | 需要 API Key | 备注 |
|--------|------------------------------|-------------|------|
| LM Studio | `lmstudio` | 否（占位符即可） | 本地模型 |
| OpenAI | `openai` | 是 | |
| Anthropic | `anthropic` | 是 | |
| Groq | `groq` | 是 | |
| Ollama | `ollama` | 否 | 本地模型 |

### 嵌入模型

默认 Full 镜像内置本地嵌入模型（BGE embedder + MiniLM cross-encoder），无需额外配置。
Slim 版需要配置外部嵌入提供商（TEI、OpenAI、Cohere 等），详见 [官方文档](https://hindsight.vectorize.io/developer/models)。

## 注意事项与 Pitfalls

- **Docker 容器访问宿主机 LM Studio**：必须用 `host.docker.internal` 而非 `localhost`，否则容器内无法连接宿主机服务。
- **大镜像拉取中断**：Full 镜像约 3.7GB (ARM64)，网络不稳定时可能 `unexpected EOF`。Docker 会缓存已下载的层，重试 `docker pull` 即可续传，无需从头开始。
- **慢网络下 Bare Metal 安装超时**：`uv pip install` 默认 HTTP 超时太短，PyPI 连接会超时。设置 `UV_HTTP_TIMEOUT=300`（5分钟）解决。国内网络下清华/阿里镜像对新包（如发布不到一周的 v0.8.5）可能返回 403（镜像未同步），需回退到 PyPI 官方源 + 长超时。
- **慢网络下的安装策略**：Full Docker 镜像 3.7GB 在慢网络下极易失败；优先尝试 Bare Metal（`uv pip install hindsight-api`），PyPI 单包下载比 GHCR 整体镜像更可靠。Slim 镜像（~500MB）是 Docker 的快速替代。
- **清华镜像同步延迟**：清华 PyPI 源对新发布包（<1周）可能返回 403，需回退到 PyPI 官方源 + `UV_HTTP_TIMEOUT=300`。检查镜像版本：`curl -s https://pypi.tuna.tsinghua.edu.cn/simple/hindsight-api/ | grep -o 'hindsight_api-[^"]*\.whl' | sort -V | uniq | tail -5`。
- **Bare Metal 依赖冲突（PYTHONPATH 污染）**：Hermes Agent 的 `PYTHONPATH` 环境变量包含 `/Users/sunwenning/.hermes/hermes-agent/venv/lib/python3.11/site-packages`，会导致 Python 3.12 的 Hindsight venv 错误加载 Python 3.11 的 pydantic_core（`ModuleNotFoundError: No module named 'pydantic_core._pydantic_core'`）。**解决方案**：启动时用 `env -u PYTHONPATH ~/hindsight-venv/bin/hindsight-api` 清除 PYTHONPATH，而非显式指定替代路径。
- **tokenizers 版本冲突**：`transformers` 要求 `tokenizers>=0.22.0,<=0.23.0`，但可能安装了 0.23.1。用 `uv pip install 'tokenizers>=0.22.0,<=0.23.0' --python ~/hindsight-venv/bin/python` 降级修复。
- **首次启动下载模型**：首次启动 Hindsight 会从 HuggingFace 下载 BGE 嵌入模型（BAAI/bge-small-en-v1.5）和重排模型（cross-encoder/ms-marco-MiniLM-L-6-v2），国内慢网络下可能需要 2-3 分钟。服务在模型下载完成后才监听端口。
- **`hindsight-local-mcp` 不是 MCP Server**：可执行文件名容易误导。它实际上调用 `hindsight_api.mcp_local.main`，只是用本地默认参数启动 `hindsight-api`。真正的 MCP Server 是 API 服务本身内置的 StreamableHTTP 端点（`/mcp/`），API 启动后自动可用。
- **sentence-transformers 已安装但 tokenizers 版本冲突**：`transformers` 要求 `tokenizers>=0.22.0,<=0.23.0`，但安装的可能是 0.23.1。报错 `ImportError: tokenizers>=0.22.0,<=0.23.0 is required` 而非 `sentence-transformers is not installed`。用 `env -u PYTHONPATH uv pip install 'tokenizers>=0.22.0,<=0.23.0' --python ~/hindsight-venv/bin/python` 降级修复（注意清除 PYTHONPATH）。
- **retain 请求体格式**：`POST /memories` 需要 `{"items": [{"content": "..."}]}` 格式，直接传 `{"content": "..."}` 会报 `{"type":"missing","loc":["body","items"],"msg":"Field required"}`。
- **Bare Metal 启动前检查**：Hindsight API 启动时需要 `python-dotenv`、`httpx` 等依赖。如果直接运行 `hindsight-api` 报 ModuleNotFoundError，需先安装所有依赖：`~/hindsight-venv/bin/python -m pip install --force-reinstall $(~/hindsight-venv/bin/python -c "import pkg_resources; d=pkg_resources.get_distribution('hindsight-api-slim'); print('\n'.join(d.requires()))" 2>/dev/null) -i https://pypi.tuna.tsinghua.edu.cn/simple`。
- **sentence-transformers 缺失**：启动时提示 `Local ML provider configured for embeddings and reranker, but 'sentence-transformers' is not installed`，需安装 `hindsight-api[local-ml]` 或改用远程嵌入提供商（如 `HINDSIGHT_API_EMBEDDINGS_PROVIDER=openai`）。
- **用户偏好：慢网络时手动下载**：当网络太慢导致自动化安装反复失败时，用户倾向于自己手动下载安装包，再由 Agent 本地安装。应在自动安装超时后主动提供 PyPI/GHCR 直链让用户手动下载。
- **Intel Mac 限制**：使用 `hindsight-all-slim` 而非 `hindsight-all`，因为完整版的本地 ML 模型（PyTorch/MLX）没有 Intel Mac wheels。
- **生产环境**：不建议使用嵌入式 pg0 数据库，应使用外部 PostgreSQL 14+ 并启用 pgvector 扩展。
- **LM Studio 模型名**：必须与 LM Studio 中实际加载的模型 ID 完全一致（通过 `curl http://localhost:1234/v1/models` 确认）。直连 llama-server 时需用完整模型路径（如 `/Users/.../model.gguf`），而非 1234 代理端口的别名（如 `ornith-1.0-35b-mtp-apex`）。
- **推荐模型：Qwen2.5-VL-7B（`qwen/qwen2.5-vl-7b`）** — 2026-08-09 起首选。它**一个模型通吃三件事**：Hindsight 事实提取（content 字段正常、JSON 稳定）+ Hermes 辅助视觉/识图（多模态）+ 其他辅助任务。仅 ~5GB，内存从 14B+VL 两个模型（15GB）降到 5GB，彻底摆脱 Gemma-4-12B 依赖。**关键兼容性**：`content` 字段正常（不像 Gemma 卡 reasoning_content）、JSON 提取稳定、中文好。切换：plist 设 `HINDSIGHT_API_LLM_MODEL=qwen/qwen2.5-vl-7b` 并重启。**注意**：7B 在超长/复杂文本的事实提取上可能略弱于 14B，大批量导入前建议小批量实测。
- **Qwen3-VL 系列不适合 Hindsight（thinking 导致超时）**：**不要**把 Hindsight 模型换成 Qwen3-VL-8B 等 Qwen3 系列。Qwen3 默认 thinking（推理）模式是双刃剑——短文本单测很快（1.4s），但 Hindsight 的**结构化 JSON 提取 prompt 复杂**，Qwen3 会深度思考导致 LLM 调用**超过 retain timeout（~120s）**，表现为 `APITimeoutError: Request timed out` 反复失败重试（实测 1.5KB 对话内容 8 小时处理不完）。同任务 VL-7B 立即完成（~10s/条）。**诊断**：单测短文本正常但 Hindsight 内超时 → 模型思考模式拖慢。`enable_thinking=false` 参数可稍快但 Hindsight 未必传。**结论**：Hindsight 用无 thinking 的 Qwen2.5-VL-7B，别用 Qwen3。
- **备选模型：Qwen2.5-14B-Instruct-1M** — 事实提取强的本地模型。中文理解强、JSON 输出稳定、输出在 `content` 字段（非 `reasoning_content`）、速度快（~4s/简单请求，94 条笔记 1-3 小时完成）。Ornith-1.0-35B 兼容但极慢（12+ 小时/94 条）。Gemma-4-12B 不兼容（`content` 为空）。切换模型后需更新 plist 中 `HINDSIGHT_API_LLM_MODEL` 并重启 Hindsight。
- **Hindsight worker 并发与本地 LLM slot 匹配**：Hindsight 默认 `max_slots=10`（10 个并发 worker），但 LM Studio 的 `--parallel 4` 只有 4 个推理 slot。批量导入时 10 个 worker 同时向 LM Studio 发送 LLM 请求，4 个 slot 瞬间占满，其余请求超时（`APIConnectionError`），导致 Hindsight 崩溃循环。**必须在 plist 中设置 worker 并发限制**：`HINDSIGHT_API_WORKER_MAX_SLOTS`（总 slot）、`HINDSIGHT_API_WORKER_RETAIN_MAX_SLOTS`（retain 专用）、`HINDSIGHT_API_WORKER_CONSOLIDATION_MAX_SLOTS`（consolidation 专用）。35B 模型推荐：`MAX_SLOTS=3, RETAIN=2, CONSOLIDATION=1`。14B 模型（如 Qwen2.5-14B）推理快可放宽：`MAX_SLOTS=5, RETAIN=4, CONSOLIDATION=1`。同时设 `HINDSIGHT_API_LLM_MAX_CONCURRENT` 和 `HINDSIGHT_API_RETAIN_LLM_MAX_CONCURRENT`（35B 设 2，14B 设 4）。
- **Slot 保留约束**：`sum(per-operation slot reservations)` 必须 `<= max_slots`。如果设 `RETAIN=2, CONSOLIDATION=1` 但 `MAX_SLOTS=2`，Hindsight 启动时会报 `ValueError: Sum of per-operation slot reservations (3: consolidation=1, retain=2) exceeds worker_max_slots (2)` 并崩溃。确保 `MAX_SLOTS >= RETAIN + CONSOLIDATION`。
- **Lantern 堵塞导致启动时 LLM 连接验证失败（服务起来但 LLM 不可用）**：Lantern 堵塞 1234 端口时，Hindsight 启动的 `Verifying connection: lmstudio/...` 会报 `Connection error`，日志出现 `LLM connection verification failed ... Server will start but LLM-dependent operations may fail until the provider is available`。此时 `/health` 仍返回 `healthy`（health 只查数据库不查 LLM），但所有 retain/consolidation 都会 `APIConnectionError` 失败。**关键诊断**：curl/httpx 独立测试 `http://127.0.0.1:1234/v1/chat/completions` 能通，但 Hindsight 进程内仍 Connection error —— 这是进程状态/连接池问题（进程是在堵塞期间启动的）。**解决**：堵塞解除后重启 Hindsight（`launchctl unload + load`），新进程会打印 `Connection verified: lmstudio/...` 证明 LLM 恢复。**另一陷阱**：堵塞期间 worker 会反复无效重试 LLM 请求，导致 LM Studio 模型一直"繁忙"（高 CPU/GPU）但无实际产出——堵塞解决、worker 空闲后模型回归空闲。
- **批量删除 async_operations 僵尸记录（payload_null）**：`pending` 状态但 `task_payload IS NULL` 的 batch_retain 是永久僵尸（无法 claim、不能 cancel，因为 cancel 只支持 pending 且 payload 已丢）。工具层无法清除，但可直接 SQL 删除：`psql` 连 pg0（凭证在 `~/.pg0/instances/hindsight/instance.json`，默认 hindsight/hindsight，端口 5433），`async_operations` 表**无外键引用**（`information_schema` 查 0 行），删除安全、不影响 `memory_units`/`chunks`/`entities`。删除前先 `\COPY ... TO <file>` 备份可回滚。**注意**：删除 `processing` 状态的操作前确认 worker 是否正在处理——有 payload 的 retain 是真实任务，删了会丢内容；只有 `payload_null` 的才可安全删。
- **直连 llama-server 绕过 1234 代理**：当 Lantern VPN 持续连接 1234 代理端口导致堵塞时，可将 Hindsight 的 `HINDSIGHT_API_LLM_BASE_URL` 改为直连 llama-server 的动态端口。从 `ps -p <llama-server-pid> -o command=` 提取 `--port`（如 61989）和 `--api-key` 参数，设 `BASE_URL=http://127.0.0.1:<port>/v1`、`API_KEY=<key>`、`MODEL=<完整模型路径>`。**注意**：端口和 API key 在 LM Studio 重启后会变，不是持久方案——优先解决 VPN 堵塞问题（关闭 VPN 或排除 1234 端口）。
- **sync_retain 批量超时**：`sync_retain` 阻塞等待 LLM 提取事实+嵌入完成，单条 10-30 秒。在 execute_code 中批量调用 94 条在 300 秒内只完成不到 10 条就超时。批量导入必须用异步 `retain`（立即返回 operation_id），再通过 `get_bank_stats` 监控 pending/processing。
- **MCP 工具未加载时的 fallback**：当 `tool_search` 找不到 hindsight 工具（如会话启动时 MCP 发现失败），可通过 curl 直接调用 MCP 端点 `/mcp/`：用 `-D` 捕获响应头获取 `Mcp-Session-Id`，再带该 header 调用 `tools/call`。
- **LM Studio 推理服务卡死（models OK 但 chat 超时）**：`/v1/models` 能正常响应，但 `/v1/chat/completions` 所有请求超时无响应（curl 10 秒后 `Operation timed out`）。这导致 Hindsight worker 全部卡在 `processing` 状态，`APIConnectionError (HTTP None)` 充满日志。诊断方法：`curl -v --max-time 10 -X POST http://localhost:1234/v1/chat/completions -H "Content-Type: application/json" -d '{"model":"MODEL_NAME","messages":[{"role":"user","content":"hi"}],"max_tokens":1}'`。如果超时，需要在 LM Studio 界面中 Stop Server → Start Server 重启推理服务。`kill` llama-server 进程也可行但需用户确认。Hindsight 的 pending 操作在 LM Studio 恢复后会自动恢复处理，无需重新提交。
- **Lantern VPN "代理所有内容"堵塞 LM Studio 端口**：Lantern VPN 的"代理所有内容"（Proxy All Content）选项会让 VPN 向所有本地监听端口发起代理连接，包括 LM Studio 的 1234 代理端口和 llama-server 的动态推理端口（如 49491/61989）。症状：`/v1/models` 能响应（轻量元数据请求能挤过），但 `/v1/chat/completions` 超时（推理请求被数百个 VPN 僵尸连接排队在前）。诊断：(1) `lsof -i :1234 | wc -l` 如果 >100 则有僵尸连接；(2) `lsof -i :1234 -c Lantern | wc -l` 确认 VPN 连接占比；(3) 从 `ps -p $(pgrep llama-server) -o command=` 提取 `--port`，直连该端口测试（如果直连能推理但 1234 不能，则确认是代理端口堵塞而非推理引擎故障）。**修复**：在 Lantern 设置中关闭"代理所有内容"选项，然后 `lsof -i :1234 | wc -l` 确认连接数降到 <20。也可在 LM Studio 界面 Stop Server → Start Server 清理已建立的僵尸连接。**关键发现**：关闭"代理所有内容"选项**可能不足以阻止 Lantern 连本地端口**——实测关闭后 Lantern 仍有 8 个 ESTABLISHED 连接到 llama-server 推理端口。如果关闭选项后连接数仍高，需要**完全退出 Lantern 应用**。Hindsight 处理完批量任务后可重新打开 Lantern。**注意**：退出 VPN 后 Hindsight 启动会遇到 HuggingFace Hub SSL 超时（嵌入模型元数据检查），Hindsight 会重试 5 次（约 5 分钟）后自动跳过。**加速方案**：在 plist 中设置 `HF_HUB_OFFLINE=1` 和 `TRANSFORMERS_OFFLINE=1` 可跳过 HF 在线检查，直接用本地缓存的嵌入模型，启动时间从 5 分钟降到 ~50 秒。
- **Hindsight 服务重启**：`launchctl kickstart -k gui/$(id -u)/com.vectorize.hindsight` 可强制重启 Hindsight LaunchAgent。重启后需等待 ~30 秒（嵌入模型加载时间）才能响应 health 检查。如果 LM Studio 推理服务仍然卡死，Hindsight 会在启动后再次因 LLM 连接失败而崩溃，形成重启循环。必须先修复 LM Studio 再重启 Hindsight。
- **Gemma-4-12B 与 Hindsight 不兼容（reasoning_content 问题）**：Gemma-4-12B（`google/gemma-4-12b-qat`）在 LM Studio 中推理时，所有输出都在 `reasoning_content` 字段（思维链/CoT），而 `content` 字段为空字符串 `""`。Hindsight 的 LLM 客户端只读取 `content` 字段，收到空响应后 worker 会一直等待/重试，表现为 `processing` 数不变但 `world`/`observation` 不增长。诊断方法：`curl -s --max-time 30 -X POST http://localhost:1234/v1/chat/completions -H "Content-Type: application/json" -d '{"model":"google/gemma-4-12b-qat","messages":[{"role":"user","content":"say ok"}],"max_tokens":5}' | python3 -c "import sys,json; r=json.loads(sys.stdin.read()); print(f'content={repr(r[\"choices\"][0][\"message\"][\"content\"])} reasoning={repr(r[\"choices\"][0][\"message\"].get(\"reasoning_content\",\"\")[:100])}')"` — 如果 `content=''` 而 `reasoning_content` 有内容，该模型不适合用于 Hindsight 的事实提取。**解决方案**：使用不在 reasoning 模式输出的模型（如 Ornith-1.0-35B-MTP-APEX），或在 LM Studio 中关闭 Gemma 的 thinking/reasoning 模式（如果该选项可用）。模型兼容性详情见 `references/hindsight-model-compatibility.md`。
- **Stuck worker 占满 slot 导致新任务无法处理**：Hindsight worker 处理 `batch_retain` 操作时如果 LLM 请求超时，worker 会进入 stuck 状态但仍占用 slot。当 stuck worker 数量等于 `MAX_SLOTS` 时，新的 `retain` 任务无法被 claim，表现为 `pending` 不下降但 `processing` 持续显示 stuck 数。日志中出现 `[STUCK?]` 标记。诊断：`tail -20 ~/Library/Logs/Hindsight/hindsight.log | grep STUCK`。解决：重启 Hindsight 释放 stuck worker（`launchctl unload` + `pkill -9` + `launchctl load`）。如果 stuck 是因为 payload_null（内容丢失的 batch_retain），这些操作永远不会完成，需要增加 `MAX_SLOTS` 让新任务有可用 slot，或等待 Hindsight 自动将 stuck 操作标记为 failed 后释放 slot。
- **batch_retain payload_null 陷阱**：Hindsight 在之前的运行中崩溃时，部分 `batch_retain` 操作的 payload（原始内容）可能丢失（数据库中 `payload=null`）。这些操作永远无法处理（`claimable=0, payload_null=26`），但仍占用 `pending` 计数。日志的 `[PENDING_BREAKDOWN]` 行会显示 `payload_null` 数量。这些无法清除（除非删除 bank 重建），只能通过增加 `MAX_SLOTS` 让新任务绕过它们，或通过 `cancel_operation` 工具取消正在 stuck 的操作。
- **PyTorch MPS 段错误（SIGSEGV）导致 Hindsight 崩溃循环**：Apple Silicon 上 Hindsight 用本地嵌入模型（bge-small）+ 重排器（MiniLM）时，若 PyTorch 自动检测到 MPS（Metal）并用它推理，Consolidation 触发重新嵌入时会在 `tensor.to('mps')` → `copy_` → `copy_cast_kernel_mps` → `MetalShaderLibrary::exec_unary_kernel` 处原生段错误（SIGSEGV/SIGBUS，EXC_BAD_ACCESS code 0x1）。崩溃是原生级，不是 Python 异常，`KeepAlive=true` 会反复重启产生大量 `Python-*.ips` 崩溃报告。诊断：`~/Library/Logs/DiagnosticReports/Python-*.ips` 里 backtrace 指向 `libtorch_cpu.dylib` 的 `at::native::mps::*`。**根因修复**：在 plist 设两个官方开关强制 CPU——`HINDSIGHT_API_EMBEDDINGS_LOCAL_FORCE_CPU=1` 和 `HINDSIGHT_API_RERANKER_LOCAL_FORCE_CPU=1`（源码 `embeddings.py`/`cross_encoder.py` 注释明写 "Force CPU mode ... to avoid MPS/XPC issues on macOS"）。性能影响≈0（bge-small 仅 33M 参数，CPU 单次嵌入几十毫秒，consolidation 大头是 LLM 调用）。**不要**用 `PYTORCH_ENABLE_MPS_FALLBACK=1` 期望救 MPS——它只回退不支持的算子，救不了 MPS 本身的崩溃，反而让 MPS 路径继续被选中。验证：重启后日志出现 `Embeddings: forcing CPU mode` 和 `Reranker: forcing CPU mode (HINDSIGHT_API_RERANKER_LOCAL_FORCE_CPU=1)`。
- **大文档导致 LM Studio 超时和 worker 卡死**：超过 10000 字符的文档在 fact extraction 阶段会产生 30000+ token 的 LLM 请求，本地模型（如 Qwen2.5-14B）处理这些请求极易超时（`APITimeoutError`），导致 worker 卡住并占满所有 retain slots。日志表现为 `[STUCK?] op=... age=7000s stage=retain_extract_facts`。**预防措施**：批量导入前先用 `list_documents` 检查文档大小分布，删除 `text_length > 10000` 的文档（`delete_document` MCP 工具）。**恢复流程**：(1) `launchctl kickstart -k gui/$(id -u)/com.vectorize.hindsight` 重启服务释放 stuck workers；(2) 删除大文档；(3) 再次重启让 pending 任务重新分配。注意 `cancel_operation` 只能取消 `pending` 状态的操作，`processing` 状态只能通过重启释放。实测阈值：>10000 字符的文档是问题源，≤5000 字符的文档处理稳定。

## 知识导入策略

### 当前策略（2026-07-31 更新）

**不再进行历史数据批量导入**。当前工作流：

1. **新对话内容** → 先沉淀到 Obsidian（50_Raw/ → 10_Knowledge_Base/）
2. **高价值信息** → 由荷妹判断后选择性导入 Hindsight
3. **判断标准**：
   - 重要决策和理由
   - 关键洞察和方法论
   - 跨会话复用的知识
   - 用户明确要求的持久化内容

**为什么不做批量导入**：
- 110 个 Obsidian 文档已导入，提取了 1435 个事实 + 27045 个链接
- 批量导入导致 stuck workers（本地 LLM 超时）
- 177 个 failed operations 无法恢复
- 选择性导入质量更高，避免噪音

### 历史批量导入参考

如需进行批量导入（不推荐），核心工作流：扫描 → 去重 → 分类打标 → 批量 `retain` → 异步监控。

**关键决策**：
- **retain vs sync_retain**：批量导入（>10条）必须用异步 `retain`，`sync_retain` 在 execute_code 中会超时（每条需 LLM 提取事实 10-30 秒，94 条在 300 秒内只完成不到 10 条）。改用 `retain` 后 94 条在 4 秒内全部提交完成。
- **直接 MCP API**：当 Hermes 会话中 MCP 工具未加载（tool_search 找不到 hindsight 工具）时，可通过 curl 直接调用 MCP 端点：初始化 session → 从响应头提取 `Mcp-Session-Id` → `tools/call` 调用具体工具。
- **大文件截断**：超过 30KB 的文件应截断，避免单条 retain 处理时间过长。
- **异步监控**：`retain` 提交后需通过 `get_bank_stats` 监控 `pending`/`processing` 数量。本地 35B LLM 处理 94 条实际耗时 **12+ 小时**（worker 并发=3，每条 5-15 分钟）。不要使用 `sync_retain` 批量调用，会在 execute_code 300 秒超时内只完成不到 10 条。
- **PYTHONPATH 污染**：通过 `source venv/bin/activate` 运行 CLI 工具会因 Hermes PYTHONPATH 污染失败，但 MCP API（curl HTTP 调用）不受影响。

详见 `references/hindsight-bulk-import.md` 获取完整工作流、代码模板和监控脚本。

## 监控导入进度

当 MCP 工具未加载时，可通过 REST API 直接查询导入状态：

```bash
# 列出所有记忆库
curl -s "http://localhost:8888/v1/default/banks" | python3 -m json.tool

# 获取记忆库详细统计
curl -s "http://localhost:8888/v1/default/banks/{bank_id}/stats" | python3 -m json.tool
```

关键字段：
- `total_nodes`: 提取的事实总数
- `total_links`: 事实间关联数
- `total_documents`: 已导入文档数
- `pending_operations`: 等待处理的操作
- `failed_operations`: 失败的操作数
- `pending_consolidation`: 等待合并的事实数
- `nodes_by_fact_type`: 按类型分布（world/observation/experience）

**进度评估**：
- 如果 `failed_operations` 占比 >20%，检查 LM Studio 稳定性或大文档超时
- 如果 `pending_consolidation` 积压，触发合并或检查 worker slot
- 如果 `pending_operations` 持续增长，检查是否有 stuck worker

**REST API vs MCP 工具**：REST API 不需要 MCP session 初始化，适合快速状态检查。MCP 工具（如 `get_bank_stats`）在会话中更方便，但需要先通过 `tool_search` 确认工具已加载。

**REST API vs MCP 工具**：REST API 不需要 MCP session 初始化，适合快速状态检查。MCP 工具（如 `get_bank_stats`）在会话中更方便，但需要先通过 `tool_search` 确认工具已加载。

## 参考文档

- 完整安装指南详见 `references/hindsight-install.md`
- MCP 集成细节、工具目录和调试方法详见 `references/hindsight-mcp-integration.md`
- 批量导入工作流（历史参考）详见 `references/hindsight-bulk-import.md`
- 运维检查与选择性导入详见 `references/hindsight-ops-check.md`
- Worker 并发调优详见 `references/hindsight-worker-tuning.md`
- LM Studio 模型兼容性详见 `references/hindsight-model-compatibility.md`
- 官方文档：https://hindsight.vectorize.io/developer/installation
- GitHub 仓库：https://github.com/vectorize-io/hindsight
