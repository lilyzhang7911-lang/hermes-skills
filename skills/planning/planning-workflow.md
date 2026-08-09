---
name: planning-workflow
description: "规划工作流统一入口：意图对齐、规格驱动、任务分解、Ticket 生成、问卷设计。"
tags: [planning, spec, tasks, tickets, questionnaire, intent-alignment]
---

# 规划工作流

统一入口：覆盖意图对齐、规格驱动开发、任务分解、Ticket 生成、问卷设计。

## 场景决策树

```
开始
├─ 需求模糊/多义？ → 意图对齐
├─ 开始新项目/特性？ → 规格驱动开发
├─ 有规格需分解？ → 任务分解
├─ 需要创建可执行任务？ → Ticket 生成
└─ 需要收集他人信息？ → 问卷设计
```

## 一、意图对齐（Intent Alignment）

### 核心原则

**目标一致 > 结构一致 > 工具与执行顺序一致**

三层对齐中，第一层（目标）最重要也最容易出错。出错根源不在于接收方理解力，而在于发出方无法一次性完整表达需求——这是人类认知的结构性限制。因此需要机制化持续校准。

### 触发条件

#### 自动触发
当识别到以下类型的需求时，**必须先启动意图对齐**：
- 任务规划、项目设计、功能开发
- 复杂决策（涉及多个选项或权衡）
- 任何需要多步骤执行的工作流
- 用户说"我想做XXX"、"帮我YYY"等模糊表达

#### 手动触发
用户明确说以下任一表达时，**强制启动**：
- "意图对齐"
- "我们对齐一下"
- "先别做，先聊聊"

#### 跳过条件（由荷妹判断）
- 纯信息查询（天气、简单事实查询）
- 任务过于简单且意图完全明确
- 用户说"直接做"、"不用了"、"跳过对齐"

### 执行流程

#### 第一步：目标澄清（Goal Clarification）

**动作：**
1. 用精炼语言复述你理解的用户意图
2. 反向确认："我的理解是XXX，对吗？"
3. 如果模糊或有多义性，主动提问缩小范围
4. 必要时使用 `clarify` 工具提供选项让用户选择

**输出格式：**
```
【目标确认】
核心目标：[一句话描述]
约束条件：[如有]
预期风格/标准：[如有]
潜在歧义点：[如有，已澄清或待确认]
```

#### 第二步：结构拆解（Structural Decomposition）

**动作：**
1. 把模糊想法拆解成原子化任务
2. 每个任务必须满足：可执行、可验证、无歧义
3. **每个任务必须映射到具体的 Skills 或 Tools**
4. 标注任务间的依赖关系和优先级

**输出格式：**
```
【结构拆解】
| 序号 | 原子化任务 | 对应Skills/Tools | 优先级 | 依赖 |
|------|-----------|-----------------|--------|------|
| 1    | [具体动作] | [skill-name/tool-name] | P0/P1/P2 | - |
| 2    | [具体动作] | [skill-name/tool-name] | P0/P1/P2 | 1 |
...

【依赖关系图】
1 → 2 → 3 (串行)
4 ←→ 5 (并行)
```

#### 第三步：执行确认（Execution Alignment）

**动作：**
1. 列出完整执行步骤和工具选择
2. 确认优先级排序
3. 标注可能的风险和备选方案
4. 等待用户明确批准后再动手

**输出格式：**
```
【执行计划】
步骤1: [具体操作] — 使用 [tool/skill]
步骤2: [具体操作] — 使用 [tool/skill]
...

【风险提示】
- [可能的风险点及应对方案]

【等待确认】
以上计划是否可行？如有调整请指出，确认后我开始执行。
```

### 关键规则

1. **禁止跳过目标澄清** — 即使你觉得意图很明确，也必须用精炼语言复述并确认
2. **每个任务必须有工具映射** — 没有对应Skills/Tools的任务不能进入执行清单
3. **用户说"直接做"时仍可提醒** — 如果识别到潜在歧义，仍应简短提示："我理解你是XXX，对吗？确认后马上开始。"
4. **对齐完成后才动手** — 除非用户明确跳过，否则不得在意图未对齐时开始执行

### 示例对话

#### 用户：我想做一个AI驱动的业务中台系统

**荷妹（启动意图对齐）：**

【目标确认】
核心目标：构建一个AI驱动的业务中台系统，为业务人员提供统一Web页面处理所有业务
约束条件：Hermes作为"四分卫"在后台协调分发任务给专业Agent
预期风格：行动导向，先开工有个抓手，反对停留在思维阶段

我的理解准确吗？有补充或修正吗？

#### 用户：对，就是这个意思

**荷妹：**

【结构拆解】
| 序号 | 原子化任务 | 对应Skills/Tools | 优先级 | 依赖 |
|------|-----------|-----------------|--------|------|
| 1 | 梳理现有业务场景和痛点 | [terminal:调研] | P0 | - |
| 2 | 设计Agent分工架构 | [intent-alignment] | P0 | 1 |
| 3 | 搭建Hermes路由分发机制 | [hermes-agent, mcp-integration] | P1 | 2 |
| 4 | 开发统一Web页面原型 | [prototype, claude-design] | P1 | 2 |

【依赖关系图】
1 → 2 → 3 → 4 (串行)
        ↘→ 4 (并行)

【执行计划】
步骤1: 调研现有业务场景 — 使用 terminal + web_search
步骤2: 设计Agent架构 — 使用 intent-alignment + delegate_task
步骤3: 搭建路由机制 — 使用 hermes-agent skill
步骤4: 开发Web原型 — 使用 prototype skill

【风险提示】
- Agent分工边界可能模糊，需要迭代明确
- Web页面技术栈需确认（React/Vue/其他）

【等待确认】
以上计划是否可行？如有调整请指出，确认后我开始执行。

## 二、规格驱动开发（Spec-Driven Development）

### 概述

在写任何代码前编写结构化规格。规格是你与人类工程师之间的共享真实来源——它定义我们构建什么、为什么、以及如何知道已完成。没有规格的代码是猜测。

### 触发条件

- 开始新项目或特性
- 需求模糊或不完整
- 变更触及多个文件或模块
- 即将做出架构决策
- 任务实现需要超过 30 分钟

**不适用：** 单行修复、拼写更正、或需求明确且自包含的变更。

### 门控工作流

规格驱动开发有四个阶段。当前阶段验证前不推进到下一阶段：

```
SPECIFY → PLAN → TASKS → IMPLEMENT
   │        │       │         │
   ▼        ▼       ▼         ▼
 Human    Human   Human     Human
 reviews  reviews reviews   reviews
```

#### Phase 1: Specify（规格化）

从高层愿景开始。向人类提问澄清问题直到需求具体。

**立即浮现假设。** 在写任何规格内容前，列出你正在做的假设：
```
ASSUMPTIONS I'M MAKING:
1. 这是一个 Web 应用（非原生移动）
2. 认证使用基于 session 的 cookies（非 JWT）
3. 数据库是 PostgreSQL（基于现有 Prisma schema）
4. 我们只针对现代浏览器（无 IE11）
→ 现在纠正我，否则我将以此 proceed。
```

不要 silently 填充模糊需求。规格的整个目的是在写代码前浮现误解——假设是最危险的误解形式。

**编写覆盖六个核心区域的规格文档：**

1. **Objective** — 我们构建什么以及为什么？谁是用户？成功是什么样的？
2. **Commands** — 完整的可执行命令带 flags，不只是工具名。
   ```
   Build: npm run build
   Test: npm test -- --coverage
   Lint: npm run lint --fix
   Dev: npm run dev
   ```
3. **Project Structure** — 源代码、测试、文档的位置。
4. **Code Style** — 一个真实代码片段展示你的风格胜过三段描述。包括命名约定、格式化规则、良好输出示例。
5. **Testing Strategy** — 什么框架、测试在哪里、覆盖率期望、哪些测试级别对应哪些关注点。
6. **Boundaries** — 三层系统：
   - **Always do:** 提交前运行测试、遵循命名约定、验证输入
   - **Ask first:** 数据库 schema 变更、添加依赖、更改 CI 配置
   - **Never do:** 提交密钥、编辑 vendor 目录、移除失败测试不经批准

**规格模板：**
```markdown
# Spec: [项目/特性名称]

## Objective
[我们构建什么以及为什么。用户故事或验收标准。]

## Tech Stack
[框架、语言、关键依赖及版本]

## Commands
[Build, test, lint, dev — 完整命令]

## Project Structure
[目录布局带描述]

## Code Style
[示例片段 + 关键约定]

## Testing Strategy
[框架、测试位置、覆盖率要求、测试级别]

## Boundaries
- Always: [...]
- Ask first: [...]
- Never: [...]

## Success Criteria
[我们如何知道已完成——具体、可测试的条件]

## Open Questions
[任何需要人类输入的未解决问题]
```

**将指令重构为成功标准。** 当收到模糊需求时，将其转化为具体条件：
```
REQUIREMENT: "让仪表板更快"

REFRAMED SUCCESS CRITERIA:
- 仪表板 LCP < 2.5s on 4G connection
- 初始数据加载在 < 500ms 内完成
- 加载期间无布局偏移 (CLS < 0.1)
→ 这些是正确的目标吗？
```

#### Phase 2: Plan（规划）

使用已验证的规格，生成技术实现计划：
1. 识别主要组件及其依赖
2. 确定实现顺序（什么必须先构建）
3. 标注风险和缓解策略
4. 识别可并行 vs 必须顺序构建的内容
5. 定义阶段间的验证检查点

> 遵循 `planning-and-task-breakdown` 获取依赖图映射和垂直切片机制；它是规范来源。

**输出约定：** 保存计划到 `tasks/plan.md`，任务列表到 `tasks/todo.md`。创建 `tasks/`（如不存在）。

#### Phase 3: Tasks（任务分解）

将计划分解为离散的、可执行的任务：
- 每个任务应在单个专注会话中完成
- 每个任务有显式验收标准
- 每个任务包括验证步骤（测试、构建、手动检查）
- 任务按依赖排序，非按感知重要性
- 没有任务应需要更改超过 ~5 个文件

**任务模板：**
```markdown
- [ ] Task: [描述]
  - Acceptance: [完成时必须为真的条件]
  - Verify: [如何确认——测试命令、构建、手动检查]
  - Files: [将触及的文件]
```

#### Phase 4: Implement（实现）

一次一个任务执行，遵循 `incremental-implementation` 和 `test-driven-development`。使用 `context-engineering` 在每个步骤加载正确的规格部分和源文件，而非用整个规格淹没代理。

### 保持规格活跃

规格是活文档，非一次性制品：
- **决策变更时更新** — 如果发现数据模型需要更改，先更新规格，再实现。
- **范围变更时更新** — 添加或裁剪的特性应在规格中反映。
- **提交规格** — 规格应与代码一起在版本控制中。
- **在 PR 中引用规格** — 链接回每个 PR 实现的规格部分。

### 反合理化表

| 合理化 | 现实 |
|---|---|
| "这很简单，不需要规格" | 简单任务不需要*长*规格，但仍需要验收标准。两行规格即可。 |
| "我会在编码后写规格" | 那是文档，不是规格。规格的价值在于在代码前强制清晰。 |
| "规格会拖慢我们" | 15 分钟规格防止数小时返工。15 分钟的瀑布胜过 15 小时的调试。 |
| "需求总会变" | 这就是规格是活文档的原因。过时的规格仍比没有规格好。 |
| "用户知道他们想要什么" | 即使清晰的请求也有隐含假设。规格浮现这些假设。 |

### 红旗

- 没有任何书面需求就开始写代码
- 在澄清"完成"意味着什么前问"我应该开始构建吗？"
- 实现任何规格或任务列表中未提到的特性
- 不记录架构决策就做出架构决策
- 因为"很明显该构建什么"而跳过规格

### 验证

- [ ] 规格覆盖所有六个核心区域
- [ ] 人类已审查并批准规格
- [ ] 成功标准具体且可测试
- [ ] 边界（Always/Ask First/Never）已定义
- [ ] 规格保存到仓库中的文件

## 三、规划与任务分解（Planning and Task Breakdown）

### 概述

将工作分解为带有明确验收标准的小而可验证的任务。好的任务分解是可靠完成工作与产生混乱之间的区别。每个任务都应该小到能在一个专注会话中实现、测试和验证。

### 触发条件

- 有规格需要分解为可执行单元
- 任务感觉太大或太模糊无法开始
- 需要在多个代理/会话间并行化工作
- 需要向人类沟通范围
- 实现顺序不明显

**不适用：** 单文件变更（范围明显），或规格已包含明确定义的任务。

### 流程

#### Step 1: 进入规划模式

在写任何代码前，以只读模式运行：
- 阅读规格和相关代码库部分
- 识别现有模式和约定
- 映射组件间的依赖关系
- 标注风险和未知项

**不要在规划期间写代码。** 输出是 `tasks/plan.md` 和 `tasks/todo.md`。

#### Step 2: 识别依赖图

映射什么依赖于什么：
```
数据库 schema
    │
    ├── API models/types
    │       │
    │       ├── API endpoints
    │       │       │
    │       │       └── Frontend API client
    │       │               │
    │       │               └── UI components
    │       │
    │       └── Validation logic
    │
    └── Seed data / migrations
```

实现顺序沿依赖图自底向上：先构建基础。

#### Step 3: 垂直切片

不要横向切片（所有数据库 → 所有 API → 所有 UI），而是每次构建一个完整的特性路径：

**错误（横向）：**
```
Task 1: 构建整个数据库 schema
Task 2: 构建所有 API endpoints
Task 3: 构建所有 UI components
Task 4: 连接一切
```

**正确（垂直）：**
```
Task 1: 用户可以创建账户 (schema + API + UI for registration)
Task 2: 用户可以登录 (auth schema + API + UI for login)
Task 3: 用户可以创建任务 (task schema + API + UI for creation)
Task 4: 用户可以查看任务列表 (query + API + UI for list view)
```

每个垂直切片交付可工作、可测试的功能。

#### Step 4: 编写任务

每个任务结构：
```markdown
## Task [N]: [简短描述性标题]

**Description:** 一段话说明此任务完成什么。

**Acceptance criteria:**
- [ ] [具体、可测试的条件]
- [ ] [具体、可测试的条件]

**Verification:**
- [ ] 测试通过: `npm test -- --grep "feature-name"`
- [ ] 构建成功: `npm run build`
- [ ] 手动检查: [验证描述]

**Dependencies:** [依赖的任务编号，或"无"]

**Files likely touched:**
- `src/path/to/file.ts`
- `tests/path/to/test.ts`

**Estimated scope:** [小: 1-2 files | 中: 3-5 files | 大: 5+ files]
```

#### Step 5: 排序和检查点

安排任务使：
1. 依赖被满足（先构建基础）
2. 每个任务让系统处于可工作状态
3. 每 2-3 个任务后出现验证检查点
4. 高风险任务前置（快速失败）

### 任务规模指南

| 规模 | 文件数 | 范围 | 示例 |
|------|--------|------|------|
| **XS** | 1 | 单函数或配置变更 | 添加验证规则 |
| **S** | 1-2 | 单组件或 endpoint | 添加新 API endpoint |
| **M** | 3-5 | 单特性切片 | 用户注册流程 |
| **L** | 5-8 | 多组件特性 | 带过滤和分页的搜索 |
| **XL** | 8+ | **太大——进一步分解** | — |

如果任务超过一个专注会话（约 2+ 小时代理工作），应进一步分解。

### 输出文件

- **Plan document:** `tasks/plan.md`
- **Task list:** `tasks/todo.md`

创建 `tasks/` 目录（如不存在）。

### 反合理化表

| 合理化 | 现实 |
|---|---|
| "我会边做边想" | 那是产生混乱和返工的方式。10 分钟规划节省数小时。 |
| "任务很明显" |  anyway 写下来。显式任务浮现隐藏依赖和遗忘的边界情况。 |
| "规划是开销" | 规划就是任务本身。没有规划的实作只是打字。 |
| "我能全部记在脑中" | 上下文窗口有限。书面计划跨越会话边界和压缩存活。 |

### 红旗

- 没有书面任务列表就开始实现
- 任务说"实现特性"但没有验收标准
- 计划中没有验证步骤
- 所有任务都是 XL 规模
- 任务间无检查点
- 不考虑依赖顺序

### 验证

- [ ] 每个任务有验收标准
- [ ] 每个任务有验证步骤
- [ ] 任务依赖被识别并按正确顺序排列
- [ ] 没有任务触及超过 ~5 个文件
- [ ] 主要阶段间存在检查点
- [ ] 人类已审查并批准计划

## 四、Ticket 生成（To Tickets）

### 概述

将计划、规格或对话分解为一组 **tickets** — tracer-bullet 垂直切片，每个声明**阻塞**它的 tickets。

### 流程

#### 1. 收集上下文

从对话上下文中已有的内容工作。如果用户传递引用（规格路径、问题编号或 URL）作为参数，获取并阅读其完整正文和评论。

#### 2. 探索代码库（可选）

如果你尚未探索代码库，这样做以理解代码的当前状态。Ticket 标题和描述应使用项目的领域词汇，并尊重你触及区域的 ADR。

寻找预重构代码以使实现更容易的机会。"使更改容易，然后做容易的更改。"

#### 3. 起草垂直切片

将工作分解为 **tracer bullet** tickets。

**垂直切片规则：**

- 每个切片穿过每一层的狭窄但**完整**路径（schema、API、UI、测试）— 垂直，不是单层的水平切片
- 完成的切片可以独立演示或验证
- 每个切片大小适合单个新鲜上下文窗口
- 任何预重构应首先完成

给每个 ticket 其**阻塞边** — 必须在其开始前完成的其他 tickets。没有阻塞者的 ticket 可以立即开始。

**宽泛重构是垂直切片的例外。** **宽泛重构**是一个机械更改 — 重命名列、重新类型化共享符号 — 其**爆炸半径**扇出到整个代码库，所以单个编辑一次破坏数千个调用点，没有垂直切片可以落地绿色。不要强制它进入 tracer bullet；将其排序为**扩展–收缩**。首先扩展：添加新形式在旧形式旁边，使nothing breaks。然后在批次中迁移调用点，批次大小由爆炸半径决定（每包、每目录），每个批次自己的 ticket 被扩展阻塞，保持 CI 绿色从批次到批次，因为旧形式仍然存在。最后收缩：一旦没有调用者剩余，删除旧形式，在 ticket 中被每个迁移批次阻塞。当即使批次也不能单独保持绿色时，保持序列但让它们共享集成分支，所有阻塞最终集成和验证 ticket — 绿色只在那里承诺。

#### 4. 询问用户

将提议的分解作为编号列表呈现。对每个 ticket，显示：

- **标题**：简短描述性名称
- **阻塞于**：哪些其他 tickets（如果有）必须先完成
- **它交付什么**：这个 ticket 使工作的端到端行为

问用户：

- 粒度感觉对吗？（太粗 / 太细）
- 阻塞边正确吗 — 每个 ticket 只依赖于真正门控它的 tickets？
- 任何 tickets 应该合并或进一步拆分吗？

迭代直到用户批准分解。

#### 5. 发布 tickets 到配置的跟踪器

发布批准的 tickets。**如何**取决于 `/setup-matt-pocock-skills` 配置的跟踪器 — tickets 相同，只有阻塞边的形状改变：

- **本地文件** → 在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md` 下为每个 ticket 写一个文件，从 `01` 开始按依赖顺序编号（阻塞者先）。每个文件的"Blocked by"列出它依赖的编号/标题。使用下面的每 ticket 文件模板 — 每个 ticket 一个文件，永远不是单个组合文件。
- **真实问题跟踪器（GitHub、Linear 等）** → 按依赖顺序（阻塞者先）为每个 ticket 发布一个问题，使每个 ticket 的阻塞边可以引用真实标识符。使用平台的原生阻塞 / 子问题关系（如果有）；否则设置每个 ticket 的"Blocked by"为阻塞问题。应用 `ready-for-agent` 分类标签，除非另有指示 — tickets 通过构造是 agent-grabbable。

工作**前沿**：任何阻塞者都已完成的 ticket。对纯线性链意味着从上到下。

不要关闭或修改任何父问题。

**本地 Ticket 模板：**

```markdown
# <NN> — <Ticket title>

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective — not a layer-by-layer implementation list.

**Blocked by:** the numbers/titles of the tickets that gate this one, or "None — can start immediately".

**Status:** ready-for-agent

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2
```

**问题模板：**

```markdown
## Parent

A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective — not layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- A reference to each blocking ticket, or "None — can start immediately".
```

在任何形式中，避免特定文件路径或代码片段 — 它们很快过时。例外：如果原型产生了比散文更精确地编码决策的片段（状态机、reducer、schema、类型形状），内联它并简要说明它来自原型。修剪到决策丰富的部分 — 不是工作演示，只是重要部分。

## 五、问卷设计（To Questionnaire）

### 概述

将用户无法单独回答的东西变成 **问卷** — 一个 Markdown 文档，他们交给一个人异步填写，或在会议上一起填写。接收者持有用户缺乏的知识；问卷将他们拉出来。

**采访发送，不是主题。** 只采访用户关于_发送_，他们总是可以回答：它发给谁，他们需要什么。文档中的问题然后针对接收者知道的和用户需要之间的**差距**。

### 流程

1. **它发给谁？** 在一次交换中询问，接收者的角色、专业知识、与用户的关系。这修复问卷的语调和它必须携带多少上下文。完成当你知道接收者是谁以及他们知道用户不知道的什么。

2. **你需要什么回来？** 在一次交换中询问，用户无法单独解决并需要从这个人的具体决策或事实。完成当你有用户必须能够做或决定的具体列表。

3. **写问卷。** 起草针对步骤 1–2 差距的问题，遵循下面的文档结构。写到当前目录的 `to-questionnaire-<slug>.md`（slug 来自主题）并报告路径。完成当文件存在且用户在步骤 2 中命名的每个项目都被问题覆盖。

### 文档结构

将文档框定为**发现问卷**：用户缺乏上下文，接收者持有它。按最重要优先排序问题 — 异步意味着你可能只得到一次通过 — 并在超过少数时用 `##` 标题按主题分组。使用下面的模板编写。

**问卷模板：**

```markdown
# <Questionnaire title>

**Purpose:** why this questionnaire exists and the decision riding on it.

**From:** <the user> — **To:** <the recipient> — **How your answers will be used:** <where they go>

## Context

One paragraph orienting a recipient who wasn't in the user's head. Enough to answer well, not a page.

## How to answer

Deadline and rough effort. Partial answers and "I don't know" are useful — flag anything you're unsure of rather than skipping it.

## <Theme heading>

One `##` section per theme. Under each, its questions, most-important-first. Every question is one idea — never compound — with an answer stub directly beneath, and a one-line _why this matters_ only where the question could be misread or invite a throwaway answer.

### What load is the system expected to handle at launch?

_Why this matters: it decides whether we provision for burst traffic now or defer it._

>

## Anything else?

A closing catch-all: anything we didn't ask that we should know?
```

## 六、常见陷阱汇总

### 意图对齐
- **跳过目标澄清** — 即使你觉得意图明确，也必须复述确认
- **没有工具映射** — 任务必须对应具体 Skills/Tools
- **未对齐就动手** — 除非用户明确跳过

### 规格驱动
- **没有书面需求就写代码** — 规格是代码前的强制清晰
- **不浮现假设** — 假设是最危险的误解形式
- **规格不活跃** — 决策变更时更新规格

### 任务分解
- **横向切片** — 使用垂直切片，每个交付可工作功能
- **任务太大** — XL 任务（8+ 文件）应进一步分解
- **没有验收标准** — 每个任务必须有可测试条件

### Ticket 生成
- **水平切片** — 使用 tracer-bullet 垂直切片
- **不标注阻塞边** — 每个 ticket 声明阻塞它的 tickets
- **宽泛重构强制垂直** — 使用扩展–收缩模式

### 问卷设计
- **采访主题不是发送** — 只采访用户关于发送（谁、什么）
- **复合问题** — 每个问题一个想法，永不复合
- **不最重要优先** — 异步意味着你可能只得到一次通过

## 七、参考资源

- planning-and-task-breakdown：任务分解和依赖图
- intent-alignment：意图对齐协议
- to-tickets：Ticket 生成和阻塞边
- to-spec：对话转规格
- to-questionnaire：问卷设计
- spec-driven-development：规格驱动开发四阶段
