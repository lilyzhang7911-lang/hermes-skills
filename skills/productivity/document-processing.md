---
name: document-processing
description: "Office 文档处理统一入口：Word/Excel/PPT/PDF 的创建、编辑、提取、转换。"
tags: [office, word, excel, ppt, pdf, document]
---

# Office 文档处理

统一入口：覆盖 Word/Excel/PPT/PDF 的创建、编辑、提取、转换。

## 场景决策树

```
开始
├─ 需要创建/编辑 Office 文档？
│  ├─ 简单文档（报告/信函）→ officecli
│  ├─ 复杂 Word（tracked changes/comments）→ docx skill
│  ├─ 复杂 Excel（财务模型/公式）→ xlsx skill
│  ├─ 复杂 PPT（morph/3D）→ powerpoint skill
│  └─ 快速生成演示文稿 → dashiai-ppt
│
├─ 需要提取/读取文档内容？
│  ├─ 文本型 PDF → markitdown 或 pymupdf
│  ├─ 扫描型 PDF（OCR）→ marker-pdf 或 markitdown+OCR
│  ├─ Word/Excel/PPT → markitdown
│  └─ 音视频/YouTube → markitdown
│
├─ 需要转换格式？
│  └─ 任意格式 → Markdown → markitdown
│
└─ 需要编辑 PDF 文本？
   └─ nano-pdf（自然语言指令）
```

## 一、创建/编辑 Office 文档

### 1.1 officecli（推荐首选）

**适用**：快速创建/编辑 docx/xlsx/pptx，无需安装 Office

**安装**：
```bash
curl -fsSL https://d.officecli.ai/install.sh | bash
```

**核心命令**：
```bash
# 创建文档
officecli create report.docx
officecli create data.xlsx
officecli create slides.pptx

# 查看结构
officecli view report.docx outline
officecli view data.xlsx stats
officecli view slides.pptx issues

# 编辑元素
officecli set report.docx '/body/p[1]' --prop text="标题"
officecli set data.xlsx '/Sheet1/A1' --prop value="Name"
officecli set slides.pptx '/slide[1]' --prop title="Q4 Report"

# 添加元素
officecli add report.docx /body --type paragraph --prop text="内容"
officecli add slides.pptx / --type slide --prop title="新页面"

# 批量操作
officecli batch data.xlsx --input updates.json
```

**三层策略**：
- **L1（读取）**：`view`、`get`、`query`
- **L2（DOM 编辑）**：`set`、`add`、`remove`
- **L3（原始 XML）**：`raw`、`raw-set`

**陷阱**：
- 路径用单引号包裹：`'/slide[1]'`（避免 shell 展开）
- 属性必须用 `--prop`：`--prop name="foo"`（不是 `--name`）
- 不确定属性名时运行 `officecli help <format> <element>`

### 1.2 Word 专项（docx skill）

**适用**：复杂 Word 文档（tracked changes、comments、TOC）

**核心工具**：
- `python-docx`：创建新文档
- `unzip` + XML 编辑：修改现有文档
- `pandoc`：读取/转换

**典型场景**：
```bash
# 创建文档
node create-docx.js

# 编辑现有文档
unzip -q doc.docx -d unpacked/
# 编辑 unpacked/word/document.xml
(cd unpacked && zip -Xr ../out.docx .)

# 添加 comments
python scripts/comment.py unpacked/ "评论内容"

# 验证
python scripts/office/validate.py out.docx --original doc.docx
```

**陷阱**：
- 不要用 `xml.etree.ElementTree`（会破坏命名空间）
- 从 unpacked 目录内 zip：`cd unpacked && zip -Xr ../out.docx .`
- Tracked changes 需要 `<w:ins>`/`<w:del>` 包裹

### 1.3 Excel 专项（xlsx skill）

**适用**：复杂 Excel（财务模型、公式、数据透视表）

**核心工具**：
- `openpyxl`：创建/编辑
- `pandas`：批量数据处理
- `recalc.py`：公式重算

**典型场景**：
```python
# 创建工作簿
from openpyxl import Workbook
wb = Workbook()
ws = wb.active
ws['A1'] = 'Name'
ws['B1'] = '=SUM(B2:B10)'
wb.save('data.xlsx')

# 读取模型（两次加载）
from openpyxl import load_workbook
wb_formulas = load_workbook('data.xlsx')  # 公式
wb_values = load_workbook('data.xlsx', data_only=True)  # 值
```

**陷阱**：
- `data_only=True` 保存会丢失公式！
- 合并单元格只写左上角
- `.xlsm` 需要 `keep_vba=True`
- 公式重算必须运行 `recalc.py`

### 1.4 PPT 专项

#### A. powerpoint skill（python-pptx）

**适用**：复杂 PPT（morph、3D、动画）

```python
from pptx import Presentation
from pptx.util import Inches, Pt

prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[0])
slide.shapes.title.text = "标题"
prs.save('slides.pptx')
```

#### B. dashiai-ppt（快速生成）

**适用**：快速生成演示文稿，12 种主题风格

**工作流**：
1. 确认主题风格（theme01-theme12）
2. 生成 `goal.json`
3. 运行渲染脚本
4. 预览/导出

```bash
# 查询页面
npm --prefix <skill-root>/project run layout:query -- --theme theme01

# 渲染
<skill-root>/scripts/render_goal_deck.sh goal.json output.html
```

#### C. dark-business-ppt-template（学术/商务）

**适用**：学术汇报、企业培训、课题展示

**配色**：深蓝黑背景（#1A1A2E）+ 青色强调（#00D9FF）+ 金色重点（#FFD700）

```python
from pptx import Presentation
from pptx.dml.color import RGBColor

prs = Presentation()
prs.slide_width = Inches(13.333)
prs.slide_height = Inches(7.5)

# 添加深色背景
fill = slide.background.fill
fill.solid()
fill.fore_color.rgb = RGBColor(0x1A, 0x1A, 0x2E)
```

## 二、提取/读取文档内容

### 2.1 markitdown（通用转换）

**适用**：任意格式 → Markdown

**安装**：
```bash
# 创建 venv（需要 Python 3.10+）
python3.11 -m venv markitdown-venv
source markitdown-venv/bin/activate
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ 'markitdown[all]'
```

**使用**：
```bash
# CLI
markitdown document.pdf -o output.md
markitdown slides.pptx > output.md

# Python API
from markitdown import MarkItDown
md = MarkItDown()
result = md.convert("document.xlsx")
print(result.text_content)
```

**支持格式**：
- PDF、DOCX、PPTX、XLSX
- 图片（EXIF + LLM Vision）
- 音频（WAV/MP3 转录）
- YouTube（字幕提取）
- HTML、CSV、JSON、XML
- EPUB

### 2.2 ocr-and-documents（PDF 专项）

**适用**：PDF 提取（文本型/扫描型）

**选择工具**：

| 特性 | pymupdf (~25MB) | marker-pdf (~5GB) |
|------|----------------|-------------------|
| 文本型 PDF | ✅ | ✅ |
| 扫描型 PDF（OCR） | ❌ | ✅ |
| 表格 | ✅（基础） | ✅（高精度） |
| 公式/LaTeX | ❌ | ✅ |
| 速度 | 即时 | 1-14s/页 |

**pymupdf**：
```bash
pip install pymupdf pymupdf4llm
python scripts/extract_pymupdf.py document.pdf --markdown
```

**marker-pdf**（需要 OCR 时）：
```bash
pip install marker-pdf
python scripts/extract_marker.py scanned.pdf
```

### 2.3 pdf-document-processing（markitdown 封装）

**适用**：企业规划、报告、论文等结构化文档

```bash
# 基本转换
markitdown input.pdf -o output.md

# 批量处理
for f in *.pdf; do markitdown "$f" -o "${f%.pdf}.md"; done
```

## 三、编辑 PDF

### 3.1 nano-pdf（自然语言编辑）

**适用**：修改 PDF 文本/错别字/标题

```bash
# 修改标题
nano-pdf edit deck.pdf 1 "Change the title to 'Q3 Results'"

# 修改日期
nano-pdf edit report.pdf 3 "Update the date from January to February"

# 修改内容
nano-pdf edit contract.pdf 2 "Change 'Acme Corp' to 'Acme Industries'"
```

**注意**：
- 页码可能是 0-based 或 1-based
- 需要 API key（LLM 驱动）
- 复杂布局修改可能需要其他方法

## 四、格式转换

### 4.1 通用转换流程

```
源文件 → markitdown → Markdown → 目标格式
```

**示例**：
```bash
# PDF → Markdown
markitdown report.pdf -o report.md

# PPT → Markdown
markitdown slides.pptx > slides.md

# Excel → Markdown
markitdown data.xlsx > data.md
```

### 4.2 特殊转换

**PDF → 图片**：
```bash
pdftoppm -jpeg -r 100 input.pdf page
```

**Word → PDF**：
```bash
python scripts/office/soffice.py --headless --convert-to pdf output.docx
```

**旧版 .doc → .docx**：
```bash
python scripts/office/soffice.py --headless --convert-to docx file.doc
```

## 五、常见陷阱汇总

### Office 通用
- 修改前关闭文件（Word/Excel/PPT）
- 路径用单引号包裹（避免 shell 展开）
- 不确定属性名时运行 `--help`

### Word
- 不要用 `xml.etree.ElementTree`（破坏命名空间）
- 从 unpacked 目录内 zip
- Tracked changes 需要特殊 XML 标签

### Excel
- `data_only=True` 保存会丢失公式
- 合并单元格只写左上角
- 公式重算必须运行 `recalc.py`

### PPT
- `shape[1]` 通常是标题占位符
- 位置索引在插入/删除后会变化
- 使用稳定 ID（`@id`）或名称（`@name`）

### PDF
- 扫描型 PDF 需要 OCR（marker-pdf）
- nano-pdf 需要 API key
- 复杂布局修改可能需要其他方法

### markitdown
- 需要 Python 3.10+（系统默认 3.9 不兼容）
- 使用 venv 安装（避免 externally-managed-environment）
- 国内镜像：`-i https://pypi.tuna.tsinghua.edu.cn/simple/`

## 六、验证清单

### 创建/编辑后
- [ ] 文件大小合理
- [ ] 用 `view` 命令检查结构
- [ ] 用 `validate` 验证（officecli）
- [ ] 渲染为 PDF 检查（Word/Excel）

### 提取后
- [ ] 内容完整（页数、章节）
- [ ] 关键信息正确提取
- [ ] 表格/图表处理正确

### 转换后
- [ ] 输出格式正确
- [ ] 内容无丢失
- [ ] 编码正确（中文）

## 七、参考资源

- officecli 帮助：`officecli help <format> <element>`
- Word 专项：`docx` skill
- Excel 专项：`xlsx` skill
- PPT 专项：`powerpoint` skill
- PDF 提取：`ocr-and-documents` skill
- 格式转换：`markitdown` skill
