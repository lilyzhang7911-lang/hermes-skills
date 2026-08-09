---
name: obsidian-hindsight-sync
description: 手动触发 Obsidian Vault 高价值内容同步到 Hindsight。只在用户明确要求时执行。
trigger: 用户说“同步 Obsidian”“同步到 Hindsight”“obsidian sync”“把 Obsidian 同步到记忆”等明确指令时触发。
---

# Obsidian → Hindsight 手动同步

将 Obsidian Vault 中高价值的结构化知识，按需同步到 Hindsight 长期记忆库。

## 适用目录

只扫描以下目录，跳过 `50_Raw/`：
- `00_Constitution/`
- `10_Knowledge_Base/`
- `20_Projects/`
- `30_Memory_Archive/Decision_Log/`
- `40_Templates_and_Skills/`

## 执行流程

### 1. 扫描变更
```bash
find "/Users/sunwenning/Desktop/MyWorkHome/My-Obsidian/Hermes Agent" \
  \( -path "*/50_Raw" -o -path "*/.obsidian" \) -prune -o \
  -name "*.md" -type f -print \
  | xargs stat -f "%m %N" 2>/dev/null \
  | sort -rn | head -20
```
只处理最近 24 小时内修改过的文件。

### 2. 读取候选内容
读取最近修改的文件内容，提取：
- 新结论 / 决策
- 跨项目通用原则
- 用户偏好变更
- 重要事实（人物、项目、模型结论）

跳过：
- 过程稿、草稿
- 仅格式更新的文件
- 模板文件（除非被修改过且有实质内容）

### 3. LLM 价值评估
对每个候选内容，评估“记忆价值”：
- 是否是新结论或决策？ (0/1)
- 是否跨项目通用？ (0/1)
- 是否影响已有工作方式？ (0/1)
- 是否包含可复用的事实/模型？ (0/1)

总分 ≥ 3 才进入下一步。

### 4. 去重检查
对通过评估的内容，先用 Hindsight recall 检查是否已存在相似记忆：
```bash
curl -s -X POST "http://127.0.0.1:8888/v1/default/banks/hermes-agent/memories/recall" \
  -H "Content-Type: application/json" \
  -d '{"query": "<候选内容>", "limit": 3}'
```
如果 Top 1 相似度 > 0.85，跳过。

### 5. 结构化提炼
将高价值内容提炼为：
```json
{
  "content": "<结构化事实，一句话>",
  "context": "<来源文件路径>",
  "tags": ["obsidian", "<分类>"]
}
```

### 6. 写入 Hindsight
```bash
curl -s -X POST "http://127.0.0.1:8888/v1/default/banks/hermes-agent/memories/retain" \
  -H "Content-Type: application/json" \
  -d '{"items": [<提炼后的JSON>]}'
```

### 7. 输出报告
将同步结果追加到：
```
/Users/sunwenning/Desktop/MyWorkHome/My-Obsidian/Hermes Agent/30_Memory_Archive/Session_Summaries/obsidian-sync-YYYY-MM-DD.md
```

报告格式：
```markdown
# Obsidian → Hindsight 同步报告

**时间**: YYYY-MM-DD HH:MM
**扫描文件数**: N
**候选内容数**: N
**通过评估**: N
**去重跳过**: N
**实际写入**: N

## 写入内容
- [content]

## 跳过内容
- [reason]
```

## 注意事项

- 本 skill **仅在用户明确要求时执行**，不自动触发
- 每次同步前先确认 Hindsight 服务健康：`curl -s http://127.0.0.1:8888/health`
- 如果 Hindsight 不可用，立即停止并报告
- 保留原始 Obsidian 文件不变，只做读取
- 如果同步过程中用户取消，立即停止，不保证原子性
