---
name: engineering/codebase-knowledge-graph
description: "使用 Graphify 将代码库转为可查询知识图谱。用于理解大型/陌生代码库、追踪调用关系、评估 PR 影响、定位核心抽象。替代盲目 grep 的结构化导航。"
license: MIT
compatibility: hermes-agent
---

# 代码库知识图谱

## 铁律

**先问结构，再问内容。** 在大型代码库中，grep 给你的是文本匹配，graphify 给你的是**关系拓扑**。当问题涉及“谁调用谁”“影响范围有多大”“核心抽象是什么”时，优先用图谱。

## 何时触发

- 进入一个**大型陌生代码库**（>5k 行或 >100 文件）
- 需要追踪**跨文件调用链**或依赖关系
- 评估变更的**影响半径**（blast radius）
- 理解**核心抽象**和模块边界
- 需要比 grep 更精确的“定义在哪里/被谁使用”查询

**何时不用**：
- 单个文件或微小改动 → 直接用 `read_file` + `search_files`
- 纯文本搜索（日志字符串、错误消息）→ grep 更快
- 首次接触代码库，尚未构建图谱 → 先跑 `/graphify update`

## 核心概念

Graphify 将代码库转为三类资产：

| 资产 | 路径 | 用途 |
|------|------|------|
| `graph.json` | `graphify-out/graph.json` | 可查询的知识图谱，节点=符号，边=关系 |
| `graph.html` | `graphify-out/graph.html` | 交互式可视化，点击探索 |
| `GRAPH_REPORT.md` | `graphify-out/GRAPH_REPORT.md` | 核心节点、意外连接、建议问题 |

**关键认知**：图谱是**增量更新**的。修改代码后，只需跑 `graphify update` 而非全量重建。

## 工作流

```
首次进入项目
    ↓
/graphify update .          # 构建/更新图谱（本地 AST 解析，零 API 调用）
    ↓
/graphify stats             # 确认图谱规模（nodes/edges/communities）
    ↓
/graphify query "..."       # 自然语言查询
/graphify explain "..."     # 解释单个节点
/graphify path "A" "B"      # 找最短路径
```

## 工具使用指南

### graphify update

构建或增量更新当前项目的知识图谱。

```bash
# 首次构建
/graphify update .

# 增量更新（代码变更后）
/graphify update .

# 强制全量重建（重构后删除了大量代码）
/graphify update . --force
```

**输出**：`graphify-out/graph.json`、`graph.html`、`GRAPH_REPORT.md`

**耗时预期**：
- 小型项目（<1k 文件）：5-15 秒
- 中型项目（1k-5k 文件）：15-60 秒
- 大型项目（>5k 文件）：1-5 分钟

### graphify query

自然语言查询知识图谱。

```bash
# 概念查询
/graphify query "auth flow"
/graphify query "database connection lifecycle"

# 精确匹配
/graphify query "UserService"
```

**最佳实践**：
- 用**名词短语**，不是完整句子
- 优先用代码中的**实际命名**（类名、函数名、模块名）
- 查询失败时，先用 `god_nodes` 看核心抽象列表，再用实际名称查询

### graphify explain

解释单个节点及其关系。

```bash
/graphify explain "APIRouter"
# 输出：定义位置、社区、度数、所有连接边（带类型和来源）
```

**适用场景**：
- 理解一个 unfamiliar class/function 的上下文
- 查看某个符号的入边和出边
- 快速判断一个节点是“核心”还是“边缘”

### graphify path

找两个概念间的最短路径。

```bash
/graphify path "FastAPI" "ModelField"
# 输出：FastAPI --uses--> DefaultPlaceholder <--references-- get_request_handler() --references--> ModelField
```

**适用场景**：
- 追踪依赖链（A 如何依赖 B）
- 理解数据流向
- 排查“为什么改了 A 会影响 B”

### graphify stats

查看图谱统计。

```bash
/graphify stats
# 输出：nodes/edges/communities 数量，帮助判断图谱是否构建成功
```

## 决策树：Graphify vs Grep vs read_file

```
需要理解代码关系？
    │
    ├─ 单个文件，找具体实现？ → read_file
    │
    ├─ 文本模式搜索（字符串、日志、配置）？ → search_files / grep
    │
    ├─ 跨文件调用链/依赖关系？ → graphify explain / path / query
    │
    ├─ 变更影响范围？ → graphify get_neighbors / get_pr_impact
    │
    ├─ 核心抽象/模块边界？ → graphify god_nodes / get_community
    │
    └─ 不确定用哪个？ → 先用 graphify stats 看图谱规模，再选
```

## Hermes 集成规范

### 自动触发规则

当用户问题符合以下模式时，**主动建议或直接使用 graphify**：

1. “这个代码库怎么实现 X？”
2. “X 依赖什么？/ 什么依赖 X？”
3. “如果我改 X，会影响哪些部分？”
4. “这个项目的核心模块是什么？”
5. “帮我理解这个 unfamiliar 代码库”

### 图谱位置约定

| 场景 | 图谱路径 |
|------|---------|
| 当前项目 | `./graphify-out/graph.json` |
| 指定项目 | `cd /path/to/project && /graphify update .` |
| Hermes 仓库 | 已预构建在 `/Users/sunwenning/.hermes/hermes-agent/graphify-out/graph.json` |

**注意**：图谱文件放在**项目目录**下，不进 `~/.hermes/`，避免污染 profile。

### 增量更新策略

```python
# 用户修改代码后，自动提示增量更新
if 用户刚执行了代码修改:
    prompt("代码已变更，建议运行 /graphify update . 刷新图谱")
```

**不要**每次查询前都更新——那会浪费 5-30 秒。只在以下情况更新：
- 首次进入项目
- 用户明确说“代码已改”
- 查询结果与预期不符（可能是图谱过时）

## 常见陷阱

### 1. 查询用自然语言而不是代码术语

```
BAD:  /graphify query "how does authentication work"
GOOD: /graphify query "auth flow"
GOOD: /graphify query "login handler"
```

### 2. 在大仓库上做全量重建

```
BAD:  每次对话都 /graphify update .
GOOD: 增量更新，必要时才 --force
```

### 3. 忽略图谱过时

```
BAD:  基于 3 天前的图谱做影响分析
GOOD: 代码变更后先 /graphify update .
```

### 4. 把 graphify 当万能工具

```
BAD:  用 graphify 找字符串 "error_code=500"
GOOD: 用 search_files 找字符串，graphify 找关系
```

## 验证清单

在标记“已理解代码库”之前：

- [ ] 已运行 `/graphify update .` 构建图谱
- [ ] 已查看 `graph_stats` 确认图谱规模合理
- [ ] 对核心概念使用过 `explain` 或 `query`
- [ ] 对关键依赖使用过 `path` 验证关系
- [ ] 如需影响分析，已确认图谱时效性（代码变更后是否更新）

## 参考资源

- Graphify 官方文档：https://graphify.net/
- Hermes MCP 配置：`hermes mcp add/remove/list/test`
- 相关 skill：`engineering/lsp-integration`（LSP 作为另一种代码导航手段）
