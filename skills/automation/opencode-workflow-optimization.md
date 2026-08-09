---
name: opencode-workflow-optimization
description: "OpenCode 工作流优化：避免上下文超限，荷妹优先直接生成结构化文档"
tags: [opencode, workflow, optimization, html-generation]
---

# OpenCode 工作流优化

## 🎯 核心原则

**荷妹优先直接生成** - HTML/CSS/JS 等结构化内容，荷妹用 `write_file` 最可靠高效。

## 📋 任务分类决策树

```
开始
├─ 简单编码任务（单文件、明确需求） → OpenCode run
├─ 复杂文档生成（多页面、详细设计） → 荷妹直接 write_file
├─ 调试/修复任务 → OpenCode interactive mode
└─ 大型项目架构 → Claude Code / 云端模型
```

## 🛠️ 执行策略

### 1. 荷妹直接生成（推荐用于 HTML/CSS/JS）

**适用场景：**
- HTML 演示文稿、网页设计
- CSS 样式文件
- JavaScript 功能模块
- 配置文件生成

**优势：**
- ✅ 避免上下文超限问题
- ✅ 完全可控的输出质量
- ✅ 无需等待 OpenCode 推理
- ✅ 支持复杂结构和详细注释

**示例：**
```python
# 荷妹直接创建 HTML 文件
write_file(path="/Users/sunwenning/Desktop/望岳.html", content=完整HTML代码)
```

### 2. OpenCode run（简单任务）

**适用场景：**
- 单文件修改
- 快速原型验证
- 简单功能添加

**提示词优化原则：**
```bash
# ❌ 复杂版本（容易超限）
opencode run "创建一个HTML演示文稿，内容是杜甫的《望岳》。要求：1) 古典水墨风格配色...2) 现代简约排版..."

# ✅ 简洁版本（避免超限）
opencode run "创建望岳.html，杜甫诗歌演示文稿，水墨风格"
```

**执行模式：**
```bash
# 一次性任务（推荐）
opencode run "task description" --model lmstudio/ornith-1.0-35b-mtp-apex

# 交互式任务（需要多轮）
opencode  # TUI 模式，后台运行
```

### 3. OpenCode interactive mode（调试修复）

**适用场景：**
- 代码调试
- Bug 修复
- 功能迭代开发

**执行方式：**
```bash
# 启动交互式会话
opencode  # background=true, pty=true

# 发送任务
process(action="submit", session_id="<id>", data="Fix the auth bug in src/auth.py")

# 监控进度
process(action="poll", session_id="<id>")
```

## ⚠️ 避免上下文超限的关键措施

### 1. 提示词精简原则
- **删除冗余描述** - "古典水墨风格配色（黑、白、灰、淡墨色）" → "水墨风格"
- **合并相似需求** - 多个 CSS 要求合并为一条
- **使用关键词而非完整句子**

### 2. 分步执行策略
```bash
# 第一步：基础结构
opencode run "创建望岳.html框架，包含封面和诗歌原文"

# 第二步：样式优化  
opencode run "为望岳.html添加水墨风格CSS动画"

# 第三步：内容完善
opencode run "补充望岳.html的赏析页和背景页"
```

### 3. 荷妹直接生成（最可靠）
对于结构化文档，荷妹直接用 `write_file` 工具：
- 完全避免 OpenCode 上下文限制
- 输出质量可控
- 执行速度快

## 📊 任务类型与推荐方式对照表

| 任务类型 | 推荐方式 | 原因 |
|---------|---------|------|
| **HTML/CSS/JS 文档** | 荷妹直接 write_file | 避免超限，完全可控 |
| **简单编码修改** | OpenCode run | 快速高效 |
| **代码调试修复** | OpenCode interactive | 需要多轮交互 |
| **大型项目架构** | Claude Code / 云端模型 | 更强推理能力 |

## 🎯 荷妹工作流中的定位

按照我们的 SOP：
- **荷妹负责：** 任务推理 + 拆解原子级子任务 + 指定执行方式
- **OpenCode 负责：** 实际编码执行（简单任务）
- **荷妹直接生成：** 结构化文档（HTML/CSS/JS）

## 🔧 验证与测试

### OpenCode 烟雾测试
```bash
opencode run "Respond with exactly: OPENCODE_SMOKE_OK"
```

### 荷妹直接生成验证
```python
# 检查文件是否成功创建
read_file(path="/Users/sunwenning/Desktop/望岳.html")
```

## 📝 常见陷阱

1. **不要过度描述** - OpenCode 提示词保持简洁
2. **避免并行复杂任务** - 串行执行更稳定
3. **及时清理会话** - 完成时 kill OpenCode 进程
4. **优先荷妹生成** - HTML/CSS/JS 文档直接用 write_file

## 🚀 最佳实践总结

1. **结构化文档 → 荷妹直接生成**（HTML/CSS/JS）
2. **简单编码 → OpenCode run**（提示词精简）
3. **调试修复 → OpenCode interactive**（多轮交互）
4. **复杂架构 → Claude Code / 云端模型**

这样既能发挥各工具的优势，又能避免上下文超限问题。