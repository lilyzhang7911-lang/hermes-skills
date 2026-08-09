---
name: obsidian-knowledge-management
description: "Obsidian 知识库统一管理：文件操作、哲学分类、三阶沉淀、关系分析。"
tags: [obsidian, knowledge-management, vault, wikilinks]
---

# Obsidian 知识库统一管理

统一入口：覆盖文件操作、架构规范、沉淀工作流、关系网络分析。

## 场景决策树

```
开始
├─ 基础读写/搜索笔记？ → 文件工具（read_file/write_file/search_files）
├─ 需要结构化查询/MCP？ → OmniRoute MCP 工具
├─ 创建新内容？ → 三阶沉淀工作流（确定放入哪一层）
├─ 分析知识图谱？ → 关系网络分析
└─ 检查架构健康？ → 健康检查清单
```

## 一、Vault 路径解析

**优先级**：环境变量 > 已知路径 > 默认路径

```bash
# 1. 检查环境变量
echo $OBSIDIAN_VAULT_PATH

# 2. 已知路径（当前用户）
/Users/sunwenning/Desktop/MyWorkHome/My-Obsidian/Hermes Agent

# 3. 默认回退
~/Documents/Obsidian Vault
```

**注意**：文件工具不展开 `$VAR`，必须先解析为绝对路径。

## 二、文件操作（基础层）

### 读取笔记
```python
read_file(path="/absolute/path/to/note.md")
```

### 列出笔记
```python
search_files(target="files", pattern="*.md", path="<vault_path>")
```

### 搜索内容
```python
# 文件名搜索
search_files(target="files", pattern="*关键词*", path="<vault_path>")

# 内容搜索
search_files(target="content", pattern="关键词", file_glob="*.md", path="<vault_path>")
```

### 创建笔记
```python
write_file(path="/absolute/path/to/new-note.md", content="# 标题\n\n内容...")
```

### 追加内容
```python
# 有稳定上下文时用 patch
patch(path="note.md", old_string="## 末尾", new_string="## 末尾\n\n新内容")

# 无稳定上下文时用 write_file 重写
```

### Wikilinks
Obsidian 用 `[[Note Name]]` 语法链接笔记。创建笔记时应在底部添加相关链接。

## 三、MCP 工具（高级层）

**适用**：结构化查询、元数据获取、文档结构映射

**前提**：OmniRoute 已运行，Obsidian API Token 已配置

| 工具 | 用途 | 关键参数 |
|------|------|---------|
| `obsidian_check_status` | 检查连接状态 | 无 |
| `obsidian_search_simple` | 简单文本搜索 | `query`, `contextLength` |
| `obsidian_search_structured` | JSON Logic 查询 | `jsonLogic` |
| `obsidian_read_note` | 读取笔记（支持 heading/block） | `path`, `targetType`, `target` |
| `obsidian_list_vault` | 列出目录树 | `path` |
| `obsidian_get_document_map` | 获取 heading 结构 | `path` |
| `obsidian_get_note_metadata` | 获取元数据 | `path` |

### 结构化查询示例
```json
{ "and": [
  { "regex": { "field": "content", "pattern": "哲学" } },
  { "eq": { "field": "path", "value": "/Hermes Agent/" } }
]}
```

## 四、五层架构规范

**核心原则**：一旦确立，永久遵循，不得随心所欲更改。

| 层级 | 文件夹 | 哲学定位 | 说明 |
|------|--------|---------|------|
| **00_Constitution** | 宪法级 | 理性 | 核心原则（≤20 文件） |
| **10_Knowledge_Base** | 知性层 | 知性 | 结构化知识（Philosophy/Concepts/Models/Methods） |
| **20_Projects** | 理性层 | 返回自身 | 项目实践 |
| **30_Memory_Archive** | 记忆归档 | 感性 | 历史记忆 |
| **40_Templates_and_Skills** | 模板技能 | 感性 | 可复用模板 |
| **50_Raw** | 感性层 | 感性 | 原始素材（30 天保留） |

### 执行规则
1. **新内容必须归入现有层级**：不允许创建新顶层目录
2. **子目录结构固定**：不得随意增删
3. **文件命名**：`YYYY-MM-DD_主题描述.md`，P0/P1/P2 标注优先级
4. **对话后必须归档**：立即归档到对应层级

### Pitfall
- **重复目录是架构退化信号**：发现 `01_Thoughts` 和 `01_哲学思考` 同时存在时，立即合并
- **新内容不应"另起炉灶"**：必须归入现有层级
- **README.md 必须同步更新**：作为架构入口，反映最新结构

## 五、哲学分类框架

### 知性层（概念 — 认知范畴）

采用**康德四大类十二范畴**分类：

| 大类 | 范畴 | 示例标签 |
|------|------|---------|
| **量** | Unit, Plurality, Measure | `quantity`, `measure` |
| **质** | Reality, Negation, Limitation | `reality`, `negation` |
| **关系** | Substance-Accident, Cause-Effect, Community | `cause-effect` |
| **模态** | Possible-Impossible, Existence, Necessity | `possibility`, `necessity` |

### 感性层（工具 — PDCA 管理）

| PDCA | 说明 | 示例 |
|------|------|------|
| **P** | 规划类工具、模板 | `Project_Template.md` |
| **D** | 执行类工具、API 配置 | `Firecrawl_配置.md` |
| **C** | 验证类工具、测试 | 审计日志 |
| **A** | 反思类工具、优化 | `Optimization_Logs.md` |

### Frontmatter 规范

```yaml
---
title: [标题]
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: rational | intellectual | sensible  # 哲学层级（必填）
category: [康德范畴标签]  # 知性层填写
pdca: P | D | C | A  # 感性层填写
status: raw | structured | active | completed | archived | 宪法
tags: [中文标签1, 中文标签2]  # 统一使用中文
---
```

### Pitfall
- **标签必须统一中文**：`philosophy` → `哲学`，`hegel` → `黑格尔`
- **知性层不应混入工具参数**：API 配置属于感性层
- **架构必须容纳反题**：`00_Constitution/Antithesis_Log.md` 记录矛盾
- **合理性要求双向链接**：通过 `[[内部链接]]` 形成知识网络

## 六、三阶沉淀工作流

### 触发条件
- 对话产生新想法/洞察
- 会议笔记、对话摘要
- 启动新项目
- 提出新概念/理论
- 项目中应用了 KB 层概念

### 第一阶：感性层（50_Raw/）

**适用**：新想法、对话记录、临时灵感

```yaml
---
title: [主题]
created: YYYY-MM-DD
source: 对话 | 会议 | 文章 | 想法
status: raw
expires: YYYY-MM-DD  # created + 30天
tags: [待分类]
---
```

**30 天内处理**：晋升/归档/删除

### 第二阶：知性层（10_Knowledge_Base/）

**晋升条件**：
- 至少被引用 2 次
- 有清晰定义和边界
- 至少 3 个 wikilinks

**子目录**：
- `Philosophy/`：哲学思考
- `Concepts/`：概念框架
- `Models/`：模型分析
- `Methods/`：方法论

```yaml
---
title: [概念名称]
type: concept | model | method | philosophy
status: structured
tags: [哲学, 黑格尔]  # 统一中文
sources: [50_Raw/...]
related: [[相关概念1]], [[相关概念2]], [[相关概念3]]
confidence: high | medium | low
---
```

### 第三阶：理性层（20_Projects/）

**适用**：启动新项目、应用 KB 概念

```yaml
---
title: [项目名称]
status: active | completed | archived
start_date: YYYY-MM-DD
applies: [[应用的概念1]], [[应用的概念2]]
outcomes: []
---
```

### 每月维护（1 日执行）
1. 检查 `50_Raw/` 过期文件
2. 决定：晋升/归档/删除
3. 执行 `hindsight sync` 同步索引

### 标签规范

**统一中文**，常用标签：
- 哲学类：哲学、黑格尔、康德、辩证法、目的论
- 技术类：AI、大模型、智能体、自动化
- 业务类：数字化转型、企业战略、本地化
- 方法论：架构、知识管理、方法论

**英文映射**：
- philosophy → 哲学
- hegel → 黑格尔
- ai → AI
- llm → 大模型
- agent → 智能体

## 七、关系网络分析

### 适用场景
- 分析笔记连接模式
- 识别核心枢纽文档
- 重构前了解知识结构
- 映射哲学 ↔ 技术层

### 分析工作流

**Step 1: 扫描所有 wikilinks**
```bash
find <vault_path> -name "*.md" -exec grep -h "\[\[.*\]\]" {} \; > /tmp/all_links.txt
```

**Step 2: 提取唯一节点和频率**
```bash
grep "\[\[" /tmp/all_links.txt | sed 's/.*\[\[\([^]]*\)\]\].*/\1/' | sort | uniq -c | sort -rn
```

**Step 3: Python 诊断（推荐，处理中文更好）**
```python
import re
from pathlib import Path

vault = Path("<vault_path>")
stats = []
for f in vault.rglob("*.md"):
    content = f.read_text(encoding="utf-8")
    links = re.findall(r'\[\[([^\]]+)\]\]', content)
    stats.append((len(links), str(f.relative_to(vault))))

stats.sort(reverse=True)
total = sum(c for c, _ in stats)
has_3plus = sum(1 for c, _ in stats if c >= 3)
orphans = sum(1 for c, _ in stats if c == 0)
print(f"Files: {len(stats)}, Links: {total}, 3+: {has_3plus}, Orphans: {orphans}")
```

### 输出模板

```markdown
## Obsidian 关系网络分析

### 📊 核心统计
- **扫描文件数**: [N]
- **有 wikilinks 的文件**: [M]
- **唯一节点数**: [K]
- **总链接引用**: [L]

### 🔗 枢纽文档（最多引用）
1. **[文档名]** ([count] 引用) — [描述]
2. ...

### 🎯 各层连接模式
[宪法层] → [哲学层] → [技术层]
    ↓           ↓           ↓
[执行层] ←→ [工具层]

### 💡 关键洞察
- [洞察 1]
- [洞察 2]

### 📝 建议
- [建议 1]
- [建议 2]
```

### 图优化工作流

**Step 1: 去重**
找到 `wiki/raw/articles/` 和主 vault 的重复文件，删除 wiki 副本。

**Step 2: 内容重组**
- 哲学文章误放 `00_Constitution/` → 移到 `10_Knowledge_Base/Philosophy/`
- 项目文件在 `20_Projects/` 根目录 → 移到项目子目录

**Step 3: Wikilink 充实**
对 KB 层 < 3 个外链的文件，添加 `## 相关概念` 章节，补充 3-5 个 wikilinks。

**Step 4: 创建 MOC**
创建 Map of Content 文件作为中间枢纽：
- `10_Knowledge_Base/Philosophy_MOC.md`
- `10_Knowledge_Base/Concepts_MOC.md`
- `10_Knowledge_Base/Models_MOC.md`
- `10_Knowledge_Base/Methods_MOC.md`

每个 MOC 有 10-30 个 wikilinks，按子主题组织。

**Step 5: 模板污染检测**
```bash
grep -r "\[\[[^]]* [0-9]\]\]" <vault_path> --include="*.md"
```
找到 `[[相关概念 1]]` 这类占位符，替换为纯文本（去掉括号）。

**Step 6: 验证**
重新运行诊断。目标：50%+ 的 KB 文件有 3+ 个外链。

## 八、健康检查清单

定期审视以下问题：

### 规范性检查
- [ ] 每个文档的 `type` 属性是否与所在层级一致？
- [ ] 文件名格式正确（`YYYY-MM-DD_主题.md`）？
- [ ] 标签统一使用中文？

### 合理性检查
- [ ] 文档间是否有双向链接？
- [ ] 是否有反题层（`Antithesis_Log.md`）？
- [ ] README.md 是否反映最新结构？

### 扩展性检查
- [ ] 是否有矛盾登记机制？
- [ ] Raw 层文件是否设置了 `expires`？
- [ ] KB 层文件是否有至少 3 个 wikilinks？

### 执行流程

**Step 1: 文件清单扫描**
```python
search_files(target="files", pattern="*.md", path="<vault_path>")
```
统计各层级分布。

**Step 2: 双向链接验证**
检查每个文档是否包含 `[[内部链接]]`，对孤立文档发出警告。

**Step 3: 命名规范检查**
遵循 `YYYY-MM-DD_主题描述.md` 格式。

**Step 4: README.md 同步更新**
作为架构入口，必须反映最新结构。

### Pitfall
- **孤立文档是架构缺陷**：没有 wikilinks 等于"并列陈列"而非"辩证关联"
- **README.md 过时导致导航失效**：必须与文件结构同步
- **定期健康检查不可省略**：架构退化等于人格退化

## 九、常见陷阱汇总

### 路径相关
- 文件工具不展开 `$VAR`，必须先解析为绝对路径
- 文件名可能有空格，用 `find ... -exec grep` 而非简单 `grep -r`
- 中文文件名需要 UTF-8 感知工具

### 架构相关
- 不允许创建新顶层目录
- 知性层不应混入工具参数（属于感性层）
- 必须维护反题登记处（`Antithesis_Log.md`）

### 标签相关
- 统一使用中文标签
- 英文标签必须转换（philosophy → 哲学）

### Wikilinks 相关
- 模板占位符 `[[相关概念 1]]` 会污染图谱，改为纯文本
- KB 层文件必须有至少 3 个 wikilinks
- Raw 层文件必须设置 `expires` 日期

## 十、参考资源

- 完整规范：`00_Constitution/三阶沉淀模型.md`
- 项目模板：`20_Projects/_Template/Project_Template.md`
- 概念模板：`40_Templates_and_Skills/Templates/Concept_Template.md`
- 语义检索：Hindsight 向量数据库（自动索引）
- 康德范畴参考：`references/kant-categories-reference.md`
- 架构执行标准：`references/architecture-standard-execution.md`
