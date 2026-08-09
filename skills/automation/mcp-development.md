---
name: mcp-development
description: "MCP 开发统一入口：构建、部署、工程化、增强（单向→全双工）。"
tags: [mcp, server, deployment, websocket, protocol-bridge]
---

# MCP 开发

统一入口：覆盖 MCP 服务器构建、部署、工程化、增强（单向→全双工）。

## 场景决策树

```
开始
├─ 需要创建新 MCP 服务器？ → 构建
├─ 需要部署到受限环境？ → 部署
├─ 需要调试/重构现有服务器？ → 工程化
└─ 需要添加双向通信？ → 增强
```

## 一、构建 MCP 服务器

### 什么是 MCP？

MCP 服务器暴露：
- **Tools**：Claude 可以调用的函数（像 API 端点）
- **Resources**：Claude 可以读取的数据（像文件或数据库记录）
- **Prompts**：预构建的提示模板

### Python MCP 服务器

#### 1. 项目设置

```bash
# 创建项目
mkdir my-mcp-server && cd my-mcp-server
python3 -m venv venv && source venv/bin/activate

# 安装 MCP SDK
pip install mcp
```

#### 2. 基础服务器模板

```python
#!/usr/bin/env python3
"""my_server.py - A simple MCP server"""

from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

# 创建服务器实例
server = Server("my-server")

# 定义工具
@server.tool()
async def hello(name: str) -> str:
    """Say hello to someone.

    Args:
        name: The name to greet
    """
    return f"Hello, {name}!"

@server.tool()
async def add_numbers(a: int, b: int) -> str:
    """Add two numbers together.

    Args:
        a: First number
        b: Second number
    """
    return str(a + b)

# 运行服务器
async def main():
    async with stdio_server() as (read, write):
        await server.run(read, write)

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

#### 3. 注册到 Claude

添加到 `~/.claude/mcp.json`：
```json
{
  "mcpServers": {
    "my-server": {
      "command": "python3",
      "args": ["/path/to/my_server.py"]
    }
  }
}
```

### TypeScript MCP 服务器

#### 1. 设置

```bash
mkdir my-mcp-server && cd my-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk
```

#### 2. 模板

```typescript
// src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-server",
  version: "1.0.0",
});

// 定义工具
server.setRequestHandler("tools/list", async () => ({
  tools: [
    {
      name: "hello",
      description: "Say hello to someone",
      inputSchema: {
        type: "object",
        properties: {
          name: { type: "string", description: "Name to greet" },
        },
        required: ["name"],
      },
    },
  ],
}));

server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "hello") {
    const name = request.params.arguments.name;
    return { content: [{ type: "text", text: `Hello, ${name}!` }] };
  }
  throw new Error("Unknown tool");
});

// 启动服务器
const transport = new StdioServerTransport();
server.connect(transport);
```

### 高级模式

#### 外部 API 集成

```python
import httpx
from mcp.server import Server

server = Server("weather-server")

@server.tool()
async def get_weather(city: str) -> str:
    """Get current weather for a city."""
    async with httpx.AsyncClient() as client:
        resp = await client.get(
            f"https://api.weatherapi.com/v1/current.json",
            params={"key": "YOUR_API_KEY", "q": city}
        )
        data = resp.json()
        return f"{city}: {data['current']['temp_c']}C, {data['current']['condition']['text']}"
```

#### 数据库访问

```python
import sqlite3
from mcp.server import Server

server = Server("db-server")

@server.tool()
async def query_db(sql: str) -> str:
    """Execute a read-only SQL query."""
    if not sql.strip().upper().startswith("SELECT"):
        return "Error: Only SELECT queries allowed"

    conn = sqlite3.connect("data.db")
    cursor = conn.execute(sql)
    rows = cursor.fetchall()
    conn.close()
    return str(rows)
```

#### Resources（只读数据）

```python
@server.resource("config://settings")
async def get_settings() -> str:
    """Application settings."""
    return open("settings.json").read()

@server.resource("file://{path}")
async def read_file(path: str) -> str:
    """Read a file from the workspace."""
    return open(path).read()
```

### 测试

```bash
# 用 MCP Inspector 测试
npx @anthropics/mcp-inspector python3 my_server.py

# 或直接发送测试消息
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | python3 my_server.py
```

### 最佳实践

1. **清晰的工具描述**：Claude 使用这些来决定何时调用工具
2. **输入验证**：总是验证和清理输入
3. **错误处理**：返回有意义的错误消息
4. **默认异步**：对 I/O 操作使用 async/await
5. **安全性**：永远不要在没有认证的情况下暴露敏感操作
6. **幂等性**：工具应该可以安全重试

## 二、部署 MCP 服务器

### 概述

自动化安装器（例如 `hermes computer-use install`）在尝试从外部源（如 GitHub）获取依赖时经常失败。这个 skill 提供手动干预和故障排除的模式。

### 模式与变通方案

#### 1. "手动脚本"模式（绕过自动化获取）

当自动化安装器因 `curl` 超时错误失败时：
- **识别源 URL**：从错误消息中提取失败的 URL（例如 `https://raw.githubusercontent.com/.../install.sh`）。
- **手动下载**：使用浏览器或手动 `curl -o <filename> <URL>` 下载脚本。
- **本地执行**：直接通过 `bash` 或 `sh` 运行下载的脚本。

#### 2. "二进制供应"模式（用户提供资产）

如果直接下载被阻止，用户可以提供预下载的资产（例如 `.zip`、`.tar.gz`）。
- **解包**：使用 `unzip` 或 `tar` 在工作目录中提取内容。
- **手动路径**：如果二进制存在但不在 `$PATH` 中，手动将它们的位置添加到 shell 配置（`.zshrc`、`.bash_profile`）或使用绝对路径。

#### 3. "环境激活"模式（Rust/Cargo）

当 `rustc` 或 `cargo` 等工具通过脚本安装但未找到时：
- **检查标准位置**：检查 `~/.cargo/bin`。
- **激活环境**：运行 `source $HOME/.cargo/env` 更新当前会话的 PATH。

#### 4. "本地模型桥接"模式（LM Studio / Ollama 作为推理后端）

当 MCP 服务器通常依赖云 AI SDK 但用户需要完全本地推理时：

1. **确认 LM Studio 正在运行**并加载目标模型——检查 `http://127.0.0.1:1234/v1/models` 返回模型列表。
2. **从源代码构建 MCP 服务器**（不是 npm）：clone/fetch → `cd packages/mcp-server/` → `npm install && npx tsc`。
3. **创建桥接脚本**（`local-pipeline.py`）调用 LM Studio 的 OpenAI 兼容端点：
   ```python
   import requests, json
   API_URL = "http://127.0.0.1:1234/v1/chat/completions"
   def call_local(prompt):
       resp = requests.post(API_URL, json={
           "model": "<model-name>",  # 例如 sc117/ornith-1.0-35b-mtp-apex-gguf
           "messages": [{"role":"user","content":prompt}],
           "max_tokens": 4096
       })
       return resp.json()["choices"][0]["message"]["content"]
   ```
4. **将桥接连接到 MCP 服务器**——用 `call_local()` 替换云 SDK 调用。服务器本身（XML 解析、验证、编辑门控）完全本地运行；只有推理路由到 LM Studio。
5. **端到端验证**：通过管道运行测试提示并确认输出匹配预期格式。

##### 陷阱
- **模型名称不匹配**：LM Studio 可能显示截断或与 GGUF 文件名不同的模型名称。在编码前总是用 `/v1/models` 验证。
- **Monorepo 依赖幽灵**（见 `mcp-server-engineering`）：从 monorepo 运行单个包需要正确链接工作区依赖——先在根目录使用 `npm install`，然后构建目标包。
- **代码中没有云 API keys**：在本地部署前审计源代码中任何硬编码的 OpenAI/Anthropic 等调用。用桥接脚本替换所有它们。

### 验证步骤

- 验证工具可用性：`which <tool_name>` 或 `<tool_name> --version`。
- 检查环境变量：`echo $PATH`。
- **本地桥接检查**：`curl -s http://127.0.0.1:1234/v1/models | python3 -m json.tool` ——应该列出加载的模型。

## 三、MCP 服务器工程化

### 核心概念

- **协议桥接**：将同步 AI 工具调用（通过 Stdio/JSON-RPC）映射到异步浏览器操作（通过 WebSockets/Events）的过程。
- **"神经桥接"模式**：在共享 `Context` 对象中使用待处理 Promise 的 `Map`，允许服务器在返回结果给 AI 之前"等待" WebSocket 响应。

### 陷阱与反模式

- **断裂的桥接（发射后不管）**：实现接收命令但缺乏将传入浏览器消息路由回触发它们的特定 MCP 工具调用的机制的传输层（像 WebSockets）。这使 AI 对执行结果"盲目"。
- **Monorepo 依赖幽灵**：尝试使用 `npx` 或直接路径从 Monorepo 运行单个包，而没有确保工作区依赖（`@repo/*`）在本地环境中正确链接/安装。
- **幽灵代理配置**：Git 卡在陈旧的、不存在的本地代理端口（例如 `127.0.0.1:49286`）由于之前的工具配置。

### 工作流

#### 维度打击（网络绕过）

当 `git clone` 因 DNS 问题或网络限制失败时：
1. 识别目标仓库和分支/提交。
2. 使用 `curl -L [URL]/archive/refs/heads/[branch].zip -o [filename].zip` 直接通过 HTTP 下载压缩存档，绕过 Git 的复杂握手。
3. 解包并在本地分析源代码。

#### 在 WebSocket 上实现请求-响应

将异步传输转换为同步工具响应：
1. **注入唯一 ID**：每条传出消息必须包含唯一 `requestId`。
2. **待处理 Map**：在应用上下文中维护 `Map<string, { resolve: Function, reject: Function }>`。
3. **Promise 包装器**：将 `send` 操作包装在新的 `Promise` 中，当通过 WebSocket 的 `on('message')` 处理器接收到匹配的 `requestId` 时解析。

## 四、MCP 增强（赋予 MCP 生命）

### 描述

分析和重构 MCP 实现的专业工作流。通过在 AI 和客户端/浏览器之间通过 WebSocket 建立双向请求-响应桥接，将"单向"基于命令的服务器转换为"全双工"智能代理。

### 触发条件

- 当 MCP 服务器被识别为具有"发射后不管"通信（没有反馈循环）时。
- 当工具调用立即返回而不等待客户端的实际执行结果时。
- 当 AI 的状态期望与浏览器的实际状态之间由于缺乏反馈而不匹配时。

### 工作流步骤

#### 1. 情报收集与解剖

- **识别传输层**：检查 `src/index.ts` 或 `package.json` 确定它使用 `StdioServerTransport` 还是基于 WebSocket 的传输。
- **检测通信间隙**：检查 `src/context.ts` 和 `src/server.ts`。寻找缺乏将 WebSocket 消息映射回 MCP 请求的传入消息处理器。

#### 2. 神经核心重建（桥接）

- **实现请求-响应映射**：在 `Context` 中实现 `Map<string, PendingRequest>`，其中 `PendingRequest` 包含 `{ resolve: Function, reject: Function }`。
- **注入唯一标识符**：修改消息负载以包含每条出站命令的唯一 `requestId`。
- **建立监听器**：在 WebSocket 连接处理器（`server.ts`）中，实现解析传入消息并在 `Context` 中解析/拒绝相应 Promise 的 `on('message')` 监听器。

#### 3. 工具实现升级（肌肉）

- **重构工具**：对 `src/tools/*.ts` 中的每个工具，用 `await context.request(type, payload)` 替换 `context.sendSocketMessage(...)`。
- **实现视觉反馈循环**：在操作后自动触发"快照"或"状态检查"，为 AI 提供结果的即时视觉确认。

#### 4. 验证与压力测试（仪式）

- **模拟三个关键路径**：
  1. **成功路径**：验证 `requestId` 匹配和 Promise 解析。
  2. **超时路径**：确保如果响应从未到达（断路器），系统不会挂起。
  3. **错误路径**：验证客户端错误是否正确传播为 AI 的异常。

### 陷阱与最佳实践

- **避免无限循环**：确保 `handleIncomingMessage` 不触发递归请求。
- **内存管理**：在解析或超时后总是从 Map 中清理/删除待处理请求，防止内存泄漏。
- **并发性**：使用唯一 ID（UUID）而不是简单计数器，避免高并发环境中的冲突。

## 五、常见陷阱汇总

### 构建
- **工具描述不清晰** ——Claude 无法决定何时调用
- **缺少输入验证** ——安全风险
- **同步 I/O** ——阻塞性能
- **无错误处理** ——难以调试

### 部署
- **自动化安装器失败** ——使用手动脚本模式
- **二进制不在 PATH** ——手动添加到 shell 配置
- **Rust 环境未激活** ——运行 `source $HOME/.cargo/env`
- **云 SDK 依赖** ——用本地模型桥接替换

### 工程化
- **Monorepo 依赖幽灵** ——先在根目录 `npm install`
- **Git 代理配置陈旧** ——清除 `http.proxy` 配置
- **网络限制** ——使用 `curl -L` 直接下载存档

### 增强
- **发射后不管** ——实现请求-响应映射
- **无限循环** ——确保 `handleIncomingMessage` 不递归
- **内存泄漏** ——清理待处理请求
- **ID 冲突** ——使用 UUID 而不是计数器

## 六、参考资源

- mcp-builder：Python/TypeScript 模板和最佳实践
- mcp-deployment：部署模式和故障排除
- mcp-server-engineering：协议桥接和请求-响应模式
- mcp-empowerment：单向→全双工转换工作流
- MCP 官方文档：https://modelcontextprotocol.io/
