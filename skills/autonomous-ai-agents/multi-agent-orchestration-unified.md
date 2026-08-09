---
name: multi-agent-orchestration-unified
description: "多 Agent 编排统一入口：决策树、编排模式、动态工作流、对抗性验证。"
tags: [multi-agent, orchestration, fan-out, delegation, verification]
---

# 多 Agent 编排

统一入口：覆盖编排决策树、编排模式、动态工作流、对抗性验证。

## 场景决策树

```
开始
├─ 任务需要多个 Agent 协作？ → 编排决策树
│  ├─ 复杂功能/重构？ → planner agent（先规划后执行）
│  ├─ 代码修改完成？ → code-reviewer + security-reviewer（并行）
│  ├─ Bug 修复/新功能？ → tdd-guide（RED→GREEN→REFACTOR）
│  ├─ 架构决策？ → architect agent
│  ├─ 构建失败？ → build-error-resolver
│  └─ 安全敏感代码？ → security-reviewer（STOP 强制拦截）
│
├─ 任务太大无法单次完成？ → 动态工作流
│  ├─ 确定性扇出（文件解析、URL 获取）？ → Layer A: execute_code
│  └─ LLM 判断扇出（分类、审查、决策）？ → Layer B: delegate_task
│
└─ 需要高质量验证？ → 对抗性收敛
   └─ N 个独立尝试 + M 个反驳者 → 只保留存活声明
```

## 一、主动编排决策树

### 触发规则总览

| 用户输入/场景 | 触发的 Agent | 执行模式 |
|--------------|-------------|---------|
| 复杂功能请求 / 重构 | **planner** | 串行（先规划后执行） |
| 刚写的代码 / 修改的代码 | **code-reviewer** + **security-reviewer** | 并行 |
| Bug 修复 / 新功能开发 | **tdd-guide** | 串行（RED→GREEN→REFACTOR） |
| 架构决策 / 系统设计 | **architect** | 串行 |
| 构建/编译失败 | **build-error-resolver** | 串行（分析→修复→验证） |
| 安全敏感代码 | **security-reviewer** | 强制拦截（STOP 流程） |
| 自主循环执行 / 监控 | **loop-operator** | 后台持续运行 |
| E2E 测试（关键用户流程） | **e2e-runner** | 串行 |
| 死代码清理 / 维护 | **refactor-cleaner** | 按需触发 |

### 编排原则

1. **不等待用户指令** — 检测到匹配场景时主动触发对应 agent
2. **并行优先** — 独立操作（如 code-review + security-review）同时执行
3. **串行依赖** — 有前后依赖的操作（如 plan → TDD → review）按序执行

### 流程图

```
用户提出需求
    ↓
[场景分类器]
    ├─ 复杂功能/重构? → planner agent
    ├─ 代码修改完成? → code-reviewer + security-reviewer (并行)
    ├─ Bug 修复/新功能? → tdd-guide (RED→GREEN→REFACTOR)
    ├─ 架构决策? → architect agent
    ├─ 构建失败? → build-error-resolver
    ├─ 安全敏感代码? → security-reviewer (STOP 强制拦截)
    └─ 其他? → 直接执行或询问用户
    ↓
[结果汇总] → 反馈给用户
```

## 二、编排模式

### 模式 1: Orchestrator-Workers

```
User Request → Orchestrator (planning, delegation)
    ├── Worker A (task 1)
    ├── Worker B (task 2)
    └── Worker C (task 3)
```

- Orchestrator 处理规划和结果聚合
- Workers 独立执行，返回结构化结果
- 使用 `delegate_task` 与 `tasks` 数组并行执行
- 为协调者 agent 设置 `role='orchestrator'`

### 模式 2: Evaluator-Optimizer

```
Generator → Evaluator (scores) → If score < threshold → regenerate
                                → Else → accept
```

- 使用 `execute_code` 进行程序化评估
- 在生成前定义清晰的评分标准
- 设置最大迭代次数防止无限循环

### 模式 3: Advisor-Orchestrator-Worker（三层模型团队）

- **Workers**：无状态生成单元，带工具（web 搜索、文件工作）
- **Advisor**：昂贵判断保持在热路径之外——策略、分解批评、风险、品味

**循环：**
1. **框架** — 陈述交付物和 3-5 个可检查的成功标准
2. **规划** — 分解为自包含子任务，最大化并行性
3. **规划审查**（强制顾问咨询 #1）
4. **委托** — 分派每个波，并行后台调用
5. **验证** — 检查每个结果，运行实际命令
6. **综合** — 组装交付物，明确解决冲突
7. **品味通过**（强制顾问咨询 #2）

**承诺边界（何时升级到顾问）：**
- 两个 worker 结果相互矛盾
- 子任务验证失败两次
- 判断调用超出成功标准
- 规划必须结构性改变

## 三、动态工作流（Plan-in-Code 扇出）

### 两个编排脚本层

| | Layer A: `execute_code` | Layer B: `delegate_task` |
|---|---|---|
| 用于 | 确定性扇出（获取 URL、解析文件） | LLM 判断扇出（分类、审查、决策） |
| 脚本持有 | 循环 + 分支 + 中间变量 | 一次性 `tasks=[...]` 数组 |
| 可用工具 | `web_search, read_file, write_file, terminal, patch` 只有 | 配置的子工具集 |
| 能调用 `delegate_task`？ | **不能** | 如果 `role='orchestrator'` |
| 成本 | 便宜（工具调用，无 LLM） | 昂贵（每子项一个模型调用树） |

**经验法则：** 先在 Layer A 做确定性部分，然后通过 Layer B 扇出不可约的 LLM 步骤。

### 同步陷阱

`delegate_task` 在父轮内**同步运行**。用户发送新消息、/stop、/new 会取消所有进行中子项。

- **前台工作流**：单轮内完成，适合分钟级扇出
- **持久工作流**：使用 kanban swarm，状态跨轮持久化

### 工作流配方

1. **分解为独立单元** — 每个单元可独立回答
2. **确定性预通过（Layer A）** — 收集清单，写到唯一目录
3. **调整扇出大小** — 每个子项 ~8-12 文件编辑或 ~50-70KB
4. **LLM 判断扇出（Layer B）** — 一个 `delegate_task` 批次
5. **在父上综合** — 读取输出文件，验证新鲜度，合并呈现

## 四、对抗性收敛

### 配方：N 个独立尝试 + M 个反驳者

1. **独立尝试（第 1 轮）** — 扇出相同问题到 N 个子项（N=2-4），不同框架/角度
2. **收集 + 去重** — 合并声明，注意协议计数
3. **反驳轮（第 2 轮）** — 每个反驳者尝试打破声明，输出 `claim_idx|survives|counter_evidence`
4. **只保留存活者** — 没有反证据的声明才浮出水面
5. **收敛（可选）** — 最多 3 轮，当无新存活声明时停止

### 为什么原子声明重要

反驳者不能打破"身份验证层有问题。"它可以打破"端点 `POST /api/users/:id/role` 在 src/routes/users.ts:142 没有角色检查。"强制特定、定位、单独可证伪的声明。

## 五、常见陷阱汇总

### 编排决策树
- **不主动触发** — 检测到匹配场景时主动触发
- **串行当可并行** — 独立操作同时执行
- **并行当有依赖** — 有依赖的操作按序执行

### 编排模式
- **上下文爆炸** — 使用 `enabled_toolsets` 限制工具访问
- **无限循环** — 设置最大迭代次数和超时守卫
- **结果聚合** — 子代理摘要是自我报告，验证后信任
- **成本管理** — 平衡代理数量和 API 成本

### 动态工作流
- **在脚本中写 delegate_task** — 不在 SANDBOX_ALLOWED_TOOLS 中
- **承诺后台/可恢复** — delegate_task 是同步的且轮范围的
- **信任 summary 字段** — 将结构化输出路由到文件
- **相同框架** — 独立尝试中变化角度

### 对抗性验证
- **非原子声明** — 强制特定、定位、可证伪的声明
- **不验证文件** — 检查输出文件计数和新鲜度
- **不报告成本** — 在提供全规模前报告范围运行的 token 成本

## 六、参考资源

- multi-agent-orchestration：基础编排模式
- advisor-orchestrator-worker：三层模型团队
- dynamic-workflow：代码化扇出工作流 + 对抗性收敛
- agent-orchestration-decision-tree：主动编排决策树
