---
name: session-search-efficiency
description: "高效使用 session_search 工具：避免重复调用、无效搜索和参数微调。当用户要求复盘/回顾历史对话时触发。"
disable-model-invocation: false
---

# Session Search Efficiency — 会话检索效率规范

## 核心原则

**一次搜索覆盖全部，不要反复试探。** 本地模型（ornith-1.0-35b）在工具调用规划上容易陷入低效循环——重复相同 query、尝试不存在的 session_id、微调无意义的参数。本 skill 提供明确的决策树和完成标准，约束这种行为。

## 触发条件

当用户提出以下意图时加载此 skill：
- "复盘" / "回顾刚才讨论了什么" / "我们之前聊过什么"
- "搜索之前的会话" / "找一下关于 XX 的记录"
- 任何需要回溯历史对话的请求

## 执行流程

### Step 1: 提取关键词（必须）

从用户请求中提取 **2-4 个核心关键词**，不要照搬原句。例如：
- "复盘 OpenClaw 安装过程" → `OpenClaw 安装`
- "之前讨论的模型对比" → `模型对比`
- "Hindsight 配置步骤" → `Hindsight 配置`

### Step 2: 单次 discover（必须）

用提取的关键词做 **一次** `session_search(query=...)`，参数：
```json
{
  "query": "<提取的关键词>",
  "limit": 5,
  "sort": "newest",
  "role_filter": "user,assistant"
}
```

**禁止在 Step 2 中指定 `session_id`。** discover 模式的目的就是找到 session_id，指定它等于跳过搜索。

### Step 3: 评估结果（必须）

检查 discover 返回的 results：
- **有匹配结果** → 进入 Step 4
- **无匹配结果** → 用更宽泛的关键词重试一次（如去掉具体名词），最多重试 1 次
- **重试仍无结果** → 直接告知用户"未找到相关会话记录"，停止搜索

### Step 4: 按需 scroll（可选）

如果 discover 返回了 session_id 且需要更详细的内容：
```json
{
  "session_id": "<从 Step 2 获得的 ID>",
  "around_message_id": <match_message_id>,
  "window": 10
}
```

**scroll 只用于读取具体内容，不用于搜索。** 不要在 scroll 结果中继续 discover。

### Step 5: 汇总输出（必须）

将搜索结果整理为结构化摘要：
- 每个相关会话的标题、时间、核心内容
- 提取关键决策和结论
- 如果用户要求"复盘"，给出连贯的叙述而非碎片化列表

## 完成标准

满足以下任一条件即停止搜索：
1. discover 返回了 ≥1 个匹配结果且已用 scroll 读取了足够上下文
2. 总共调用 `session_search` ≤ 3 次（discover + 可选重试 + 可选 scroll）
3. 用户明确表示"够了"或话题已结束

## 禁止行为

以下行为 **严格禁止**，违反即视为低效搜索：

| 禁止项 | 原因 |
|--------|------|
| 对相同 query 重复调用 discover | 浪费次数，结果不变 |
| 指定不存在的 session_id | 返回 error，白白消耗迭代 |
| 在 discover 中微调 window/role_filter 等参数 | 不影响搜索结果质量 |
| 连续调用 ≥4 次 session_search | 接近系统上限，风险高 |
| 用完整句子做 query（如"帮我安装 OpenClaw"） | FTS5 对长句匹配差，应提取关键词 |

## 反模式示例

### ❌ 低效搜索（本 skill 要阻止的）
```
调用1: session_search(query="OpenClaw 安装 LM Studio ornith 本地模型", ...)
调用2: session_search(query="OpenClaw 安装 LM Studio ornith 本地模型", session_id="xxx")  # 不存在的 ID
调用3: session_search(query="OpenClaw 安装 LM Studio ornith 本地模型", ...)  # 重复
调用4: session_search(query="OpenClaw 安装 LM Studio ornith 本地模型", session_id="yyy")  # 又是不存在的 ID
```

### ✅ 高效搜索（本 skill 要求的行为）
```
调用1: session_search(query="OpenClaw 安装", limit=5, sort=newest)  → 找到 session A
调用2: session_search(session_id=A, around_message_id=X, window=10)  → 读取详细内容
输出: 结构化摘要
```

## 关键词提取技巧

| 用户原话 | 推荐关键词 | 原因 |
|----------|-----------|------|
| "复盘 OpenClaw 安装过程" | `OpenClaw 安装` | 去掉动词，保留核心名词 |
| "之前讨论的模型对比结论" | `模型对比` | "讨论""结论"是噪音词 |
| "Hindsight 配置步骤和失败模式" | `Hindsight 配置` | 合并为单一概念 |
| "我们聊过什么关于业务中台的" | `业务中台` | 直接提取主题词 |

## 与本地模型的特别约定

ornith-1.0-35b-mtp-apex 在以下场景容易失控：
1. **重复搜索** — 模型倾向于认为"再试一次可能不同"，但 session_search 的 FTS5 引擎对相同 query 返回确定性结果
2. **无效 session_id** — 模型会编造或猜测 session_id，应先用 discover 获取真实 ID
3. **参数微调执念** — 模型喜欢调整 window/limit/role_filter 等参数，但这些不影响核心搜索结果

本 skill 通过明确的"最多 3 次调用"硬上限和禁止列表来约束这些行为。
