---
name: omniroute-routing-patterns
description: "OmniRoute 路由策略与压缩引擎的哲学映射和工程模式参考。用于理解多 Provider 自动降级、17+ 路由策略、Fusion 合成、RTK/Caveman 压缩等设计模式，并将其映射到 Hermes 的多模型协作场景。"
category: research
version: 1.0.0
lastUpdated: 2026-07-13
---

# OmniRoute Routing Patterns — 哲学与工程参考

## 核心洞察

OmniRoute 是一个 **Mirror of Thought** 的工程化实现：它不产生智能，而是通过路由策略确保每次调用命中最合适的智能源。

## 路由策略映射（思维通道选择）

### 17+ 种策略的哲学含义

| 策略 | 工程行为 | 哲学映射 |
|---|---|---|
| `priority` | 按优先级顺序尝试，耗尽后进入下一个 | 黑格尔正题→反题的有序遍历 |
| `weighted` | 按权重随机选择 | 概率性思维通道选择 |
| `round-robin` | 循环使用所有目标 | 均衡分配思维负载 |
| `fusion` 🧬 | 并行扇出到多个 model，judge 综合为唯一答案 | **多镜合成** — 最接近"集体智慧"的路由模式 |
| `context-relay` | 跨 target 传递上下文（长对话） | 记忆连续性保障 |
| `cost-optimized` | 最小化每请求成本 | 资源最优分配 |
| `headroom` | 选择剩余配额最多的目标 | 优先使用充裕资源 |
| `lkgp` | Last-Known-Good — 粘性路由到上次成功的 target | 路径依赖/习惯保持 |
| `auto` | 9 因子实时评分，动态选择最优 provider | **动态感知** — 最接近"思维之镜"的自适应 |

### Auto-Combo 12 因子评分体系

所有权重之和 = 1.0：

| 因子 | 默认权重 | 说明 |
|---|---|---|
| health | 0.20 | 断路器健康分 |
| quota | 0.15 | 剩余配额/速率限制余量 |
| costInv | 0.15 | 反向成本（60% input + 40% output） |
| latencyInv | 0.12 | 反向 p95 延迟 |
| taskFit | 0.08 | 任务类型适配度 |
| stability | 0.05 | 方差稳定性 |
| tierPriority | 0.05 | 账户层级优先级 |
| tierAffinity | 0.05 | 候选层级与推荐层级的亲和度 |
| specificityMatch | 0.05 | 请求具体性与模型层级匹配 |
| contextAffinity | 0.05 | 上下文窗口需求与模型窗口亲和度 |
| connectionDensity | 0.05 | 同 Provider 多连接负载分散 |

**Mode Packs**（预定义权重配置）：

| Pack | 核心偏好 | 适用场景 |
|---|---|---|
| ship-fast | latencyInv 0.32 + health 0.28 | 实时交互、autocomplete |
| cost-saver | costInv 0.37 | 批量处理、后台任务 |
| quality-first | taskFit 0.37 + stability 0.15 | 代码生成、复杂推理 |
| offline-friendly | quota 0.37 + health 0.28 | 离线/弱网环境 |

## Fusion 策略（多镜合成）

`fusion` 是最接近"集体智慧"的路由模式：

1. **Fan-out** — prompt 并行发送给所有 panel model
2. **Quorum-grace collection** — 达到 minPanel 答案后启动 grace 计时器
3. **Judge synthesis** — panel 答案匿名化，judge 分析共识/矛盾/盲区，写出权威答案
4. **Graceful degradation** — 0 个 panel → 503；仅 1 个幸存者 → 直接返回

配置参数：
- `config.judgeModel` — 综合者模型（默认第一个 panel model）
- `config.fusionTuning.minPanel` — 最少成功答案数（默认 2）
- `config.fusionTuning.stragglerGraceMs` — 落后者宽限时间（默认 8000ms）
- `config.fusionTuning.panelHardTimeoutMs` — 绝对超时上限（默认 90000ms）

## RTK + Caveman 压缩引擎（净化）

### 10 个可组合压缩引擎

| 引擎 | 机制 | 典型节省 |
|---|---|---|
| RTK (Recursive Token Knowledge) | 递归压缩已知信息 | 15-40% |
| Caveman | 去重 + ultra 压缩规则包（支持多语言） | 30-95% |
| LLMLingua-2 ONNX | SLM 语义压缩 | 20-60% |
| Ultra (heuristic/SLM two-tier) | 双层启发式+轻量模型 | 自适应 |
| Anthropic Context Editing | 委托压缩 | 按需 |

### 关键设计模式

- **Inflation Guard**: 压缩后比原文更长时，丢弃压缩结果，发送原始内容
- **Compression Studios**: 可视化调试 + A/B Compare
- **Adaptive context-budget dial**: 只在必要时升级压缩级别
- **Per-request control**: `x-omniroute-compression` header 控制单请求压缩行为

## 三层韧性架构（自我修复）

| 层级 | 作用域 | 机制 |
|---|---|---|
| Circuit Breaker | 整个 Provider | OPEN → 停止锤击；HALF_OPEN → probe；CLOSED → 正常 |
| Connection Cooldown | 单个账户/密钥 | 跳过被限流的密钥，其他继续服务 |
| Model Lockout | Provider + 模型 | 仅隔离配额耗尽的单一模型 |

**自愈机制**:
- Score < 0.2 → 排除 5 分钟（progressive backoff, max 30 min）
- Incident mode: >50% OPEN → 禁用探索，最大化稳定性
- Cooldown recovery: 排除后首次请求为 probe（降低超时）

## 对 Hermes 的参考价值

### 路由策略应用

当 Hermes 需要多模型协作时，可参考 OmniRoute 的策略：

1. **`auto` / `auto/coding`** — 动态感知任务类型，选择最优本地/远程模型
2. **`fusion`** — 并行发送到多个本地 LLM，judge 综合（适合复杂推理）
3. **`context-relay`** — 长对话中跨模型传递上下文
4. **`lkgp`** — 粘性路由到上次成功的模型（保持会话连续性）

### 压缩应用

- 对工具输出（git diff, grep, logs）使用 Caveman 规则包去重
- 对多轮对话使用 RTK 递归压缩已知信息
- 使用 Inflation Guard 避免压缩反而增加 token

### 韧性应用

- LM Studio 连接失败时，Circuit Breaker 自动隔离
- 多 Provider 场景下，Connection Cooldown 确保其他密钥继续服务
- Model Lockout 仅隔离特定模型的配额问题

## Pitfalls

1. **不要盲目信任压缩** — Inflation Guard 是必须的，压缩可能使 prompt 变长
2. **Fusion 策略成本高** — 并行扇出 + judge 综合 = N+1 次 API 调用，仅适合重要决策
3. **Quota-Share 需要上游支持** — 配额共享依赖提供商的 token usage headers
4. **Auto-Combo 是虚拟的** — 不持久化到 DB，每次请求重新构建候选池

## Related

- [[OmniRoute-Philosophical-Analysis]] — 哲学框架映射分析（Obsidian note）
- OmniRoute source: https://github.com/diegosouzapw/OmniRoute
- Docs: `docs/routing/AUTO-COMBO.md`, `docs/compression/COMPRESSION_ENGINES.md`
