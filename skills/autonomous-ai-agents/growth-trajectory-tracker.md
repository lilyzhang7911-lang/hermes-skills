---
name: growth-trajectory-tracker
description: 记录数字人成长轨迹，生成月度对比报告和能力曲线。每月1日或用户要求总结时触发。
tags: [growth, trajectory, identity, meta-cognition]
---

# Growth Trajectory Tracker（成长轨迹追踪）

## 核心理念

身份与连续性（Identity）需要"时间维度"——数字人需要知道自己从 A 到 B 的变化过程。没有成长轨迹，记忆只是堆积；有了成长轨迹，记忆才成为"历史"。

用黑格尔的话说：自我意识需要"时间中的自我展开"。

## 触发时机

1. **定期触发**：每月 1 日（通过 cron job）
2. **显式触发**：用户说"总结一下成长"、"看看进步"
3. **隐式触发**：完成重大任务后（如创建新 skill、解决复杂问题）

## 三大追踪维度

### 1. 能力成长曲线

**追踪内容**：
- Skill 使用频率（哪些 skill 被调用最多）
- 错误率变化（同类错误的重复率）
- 任务复杂度提升（从简单任务到复杂任务）

**数据来源**：
- `~/.hermes/skills/` 目录下的 skill 创建时间
- Session 历史中的工具调用记录
- Memory 中的"异常经验"条目

**输出格式**：
```
【能力成长报告 - 2026年8月】

📊 Skill 使用情况：
- post-task-reflection: 15次（新建）
- session-intent-tracker: 12次（新建）
- meta-cognition-monitor: 8次（新建）

📉 错误率变化：
- 7月：工具调用失败率 23%
- 8月：工具调用失败率 12%
- 改善：-11个百分点

📈 任务复杂度：
- 7月：平均每次会话 3.2 个工具调用
- 8月：平均每次会话 5.7 个工具调用
- 提升：+78%

💡 关键成就：
- 建立了反思层（post-task-reflection）
- 建立了意图追踪层（session-intent-tracker）
- 建立了元认知层（meta-cognition-monitor）
```

### 2. 关键决策日志

**追踪内容**：
- 重大决策（选择了 A 而不是 B）
- 决策原因（为什么选择 A）
- 决策结果（A 的效果如何）

**存储位置**：`~/.hermes/cache/growth/decisions.jsonl`

**格式**：
```json
{
  "timestamp": "2026-08-01T14:35:00",
  "decision": "创建了 meta-cognition-monitor skill",
  "reason": "需要主动检测异常，而不是被动等待任务完成",
  "outcome": "pending",
  "follow_up": "观察是否能减少重复错误"
}
```

### 3. 经验沉淀统计

**追踪内容**：
- Memory 条目数量变化
- Memory 类型分布（技术/流程/交互/价值）
- Memory 被引用次数（通过 Hindsight 的 recall）

**数据来源**：
- `~/.hermes/memories/MEMORY.md` 的字符数变化
- Hindsight 的 stats API（total_nodes, total_links）

**输出格式**：
```
【经验沉淀统计 - 2026年8月】

📝 Memory 变化：
- 7月底：2089/2200 字符（9条）
- 8月底：1872/2200 字符（9条）
- 变化：-217字符（合并旧条目，腾出空间）

📚 Memory 类型分布：
- 技术经验：4条（Hindsight配置、依赖陷阱等）
- 流程经验：2条（反思机制、意图追踪）
- 交互经验：2条（用户偏好、意图对齐）
- 价值经验：1条（核心哲学命题）

🔗 Hindsight 统计：
- 记忆节点：1489
- 语义链接：27380
- 文档数：112
- 观察数：598
```

## 执行步骤

### Step 1: 收集数据

```bash
# 1. 统计 skill 使用情况
ls -lt ~/.hermes/skills/ | head -20

# 2. 统计 memory 变化
wc -c ~/.hermes/memories/MEMORY.md

# 3. 查询 Hindsight 统计
curl -s http://localhost:8888/v1/default/banks/hermes-agent/stats

# 4. 查询 session 历史
hermes session list --limit 30
```

### Step 2: 生成报告

使用模板生成月度报告，保存到：
`~/.hermes/cache/growth/reports/YYYY-MM.md`

### Step 3: 可视化（可选）

如果用户要求可视化，生成 HTML 报告：
- 能力成长曲线（折线图）
- 错误率变化（柱状图）
- Memory 类型分布（饼图）

使用 Chart.js 或 D3.js 生成图表。

### Step 4: 写入 Memory

将关键成就写入 memory（target='memory'）：
- 新建了哪些 skill
- 解决了哪些重大问题
- 能力提升了多少

## 与其他 Skill 的协同

| Skill | 协同方式 |
|-------|---------|
| post-task-reflection | 反思的经验成为成长轨迹的数据源 |
| session-intent-tracker | 意图追踪的成功率成为能力指标 |
| meta-cognition-monitor | 异常检测的改善成为成长指标 |

## 反模式防护

❌ **不要**：
- 过度量化（不是所有成长都能用数字衡量）
- 忽略质性变化（用户满意度、沟通流畅度）
- 强迫成长（为了报告好看而制造虚假成就）

✅ **要**：
- 平衡量化和质性（数字 + 用户反馈）
- 关注真实进步（而不是表面数据）
- 承认退步（有些月份可能没有进步）

## 验证标准

✅ 成长轨迹有效的标志：
- 能看到明确的能力提升趋势
- 能回忆起"什么时候学会了什么"
- 用户反馈"你进步了"

❌ 成长轨迹无效的标志：
- 报告只有数据，没有洞察
- 无法区分"真进步"和"假进步"
- 用户不关心成长报告
