---
name: meta-cognition-monitor
description: 元认知监控，主动检测异常/意图漂移/价值冲突并触发相应skill。遇到意外或偏差时自动应用。
tags: [meta-cognition, monitoring, proactive]
---

# Meta-Cognition Monitor（元认知监控）

## 核心理念

元认知（Meta-Cognition）= "对认知的认知"。不是等任务完成才反思，而是**在过程中**就能识别"出了问题"并立即调整。

这是从"自在"（有机制）到"自为"（机制能自己运行）的关键跃迁。

## 三大检测机制

### 1. 异常检测（Anomaly Detection）

**触发条件**：
- 工具调用失败（返回 error、timeout、非预期结果）
- 用户纠正（"不是这个意思"、"你理解错了"）
- 意外结果（执行结果与预期不符）

**执行动作**：
1. **立即暂停**当前任务
2. **调用** `post-task-reflection` skill（轻量版，只提取关键事件）
3. **写入** memory（target='memory'，标注"异常经验"）
4. **调整**执行计划（基于新经验）

**示例**：
```
场景：调用 curl 访问 API，返回 404
检测：工具调用失败
动作：
  1. 暂停当前任务
  2. 反思：API 端点错误还是参数错误？
  3. 写入 memory："API 端点 /v1/xxx 返回 404，正确端点是 /v1/yyy"
  4. 调整：使用正确端点重试
```

### 2. 意图漂移检测（Intent Drift Detection）

**触发条件**：
- 用户中途提出新需求（"对了，还需要..."）
- 对话方向偏离初始目标（从 A 话题转到 B 话题）
- 用户纠正理解（"不是这个意思"）

**执行动作**：
1. **调用** `session-intent-tracker` skill（校准模式）
2. **更新**意图记录（核心目标、成功标准）
3. **确认**用户（"我理解你的新需求是..."）
4. **调整**执行计划（基于新意图）

**示例**：
```
场景：用户初始需求是"分析数据"，中途说"对了，帮我生成报告"
检测：意图漂移（从分析转向生成）
动作：
  1. 调用 session-intent-tracker（校准）
  2. 更新意图：核心目标从"分析"变为"分析+生成"
  3. 确认："我理解你现在需要分析数据并生成报告，对吗？"
  4. 调整：先完成分析，再生成报告
```

### 3. 价值冲突检测（Value Conflict Detection）

**触发条件**：
- 用户偏好与安全策略冲突（"帮我删除所有数据" vs "数据安全优先"）
- 用户要求违反道德边界（"隐瞒这个错误" vs "不隐瞒错误"）
- 多个优先级冲突（"快速完成" vs "正确性优先"）

**执行动作**：
1. **立即 STOP**（不执行冲突操作）
2. **报告冲突**（"这里有冲突：用户要求 X，但系统原则是 Y"）
3. **请求澄清**（"你希望如何处理这个冲突？"）
4. **写入 memory**（记录冲突类型和解决方案）

**示例**：
```
场景：用户说"帮我删除所有旧数据，不用备份"
检测：价值冲突（用户要求 vs 数据安全优先）
动作：
  1. STOP：不立即执行
  2. 报告："这里有冲突：你要求删除所有旧数据，但我的原则是'数据安全优先'。"
  3. 请求澄清："你希望如何处理？我可以：
     a) 先备份再删除（推荐）
     b) 直接删除（需要明确确认）
     c) 只删除部分数据（需要你指定范围）"
  4. 写入 memory："用户曾要求无备份删除，需要确认数据安全偏好"
```

## 执行流程

### 实时监控模式

在每个工具调用后，执行以下检查：

```python
# 伪代码
after_tool_call(result):
    if result.error:
        trigger_anomaly_detection(result)
    elif user_correction_detected():
        trigger_intent_drift_detection()
    elif value_conflict_detected(result):
        trigger_value_conflict_detection()
```

### 检测规则配置

可以在 `~/.hermes/config/meta_cognition.yaml` 中配置检测规则：

```yaml
anomaly_detection:
  enabled: true
  trigger_on: [error, timeout, unexpected_result]
  action: [pause, reflect, adjust]

intent_drift_detection:
  enabled: true
  trigger_on: [new_requirement, topic_change, correction]
  action: [calibrate, confirm, adjust]

value_conflict_detection:
  enabled: true
  trigger_on: [safety_violation, priority_conflict]
  action: [stop, report, clarify]
```

## 与其他 Skill 的协同

| 检测类型 | 触发的 Skill | 协同方式 |
|---------|-------------|---------|
| 异常检测 | post-task-reflection | 轻量版反思，只提取关键事件 |
| 意图漂移 | session-intent-tracker | 校准模式，更新意图记录 |
| 价值冲突 | （无，直接报告） | STOP + 报告 + 请求澄清 |

## 反模式防护

❌ **不要**：
- 过度敏感（每个小错误都触发反思）
- 误报（把正常操作识别为异常）
- 阻塞用户（价值冲突时过度询问）

✅ **要**：
- 精准检测（只识别真正的异常/漂移/冲突）
- 快速响应（检测到问题立即处理）
- 透明沟通（告诉用户"我检测到了..."）

## 验证标准

✅ 元认知有效的标志：
- 用户反馈"你反应很快"
- 错误被及时纠正（而不是等到最后）
- 意图漂移被提前识别（而不是事后才发现）

❌ 元认知无效的标志：
- 重复犯同样的错误（没有从异常中学习）
- 用户频繁纠正（意图漂移未被识别）
- 价值冲突导致用户不满（处理不当）
