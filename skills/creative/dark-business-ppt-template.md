---
name: dark-business-ppt-template
description: 深色商务PPT模板 — 简约大气风格，适用于学术汇报、企业培训、课题展示。基于python-pptx生成，16:9宽屏。
tags: [ppt, template, business, academic]
created: 2026-07-15
---

# 深色商务PPT模板 v1

## 触发条件
用户要求制作学术汇报、课题展示、企业培训类PPT，且偏好简约大气风格时。

## 配色方案（固定）
```python
BG_DARK      = RGBColor(0x1A, 0x1A, 0x2E)   # 深蓝黑背景
BG_CARD      = RGBColor(0x24, 0x24, 0x3E)    # 卡片背景
ACCENT_CYAN  = RGBColor(0x00, 0xD9, 0xFF)    # 主强调（标题、序号）
GOLD         = RGBColor(0xFF, 0xD7, 0x00)     # 副强调（重点、引用）
WHITE        = RGBColor(0xFF, 0xFF, 0xFF)     # 正文
LIGHT_GRAY   = RGBColor(0xCC, 0xCC, 0xCC)    # 次要文字
MEDIUM_GRAY  = RGBColor(0x99, 0x99, 0x99)    # 注释/时间线
```

## 字体规范
- 中文字体: Microsoft YaHei（微软雅黑）
- 标题: 28-36pt 加粗
- 正文: 14-18pt
- 注释: 12-14pt

## 版式模板库

### 封面页
- 居中大标题（36pt白色加粗）
- 副标题（20pt青色）
- 金色装饰线分隔
- 底部单位信息（18pt浅灰居中）

### 目录页
- "目 录"大标题 + 青色短横线
- 编号（24pt青色加粗）+ 章节名（18pt白色）纵向排列，每项间距0.75英寸

### 双栏内容页
- 顶部章节标题（28pt青色加粗）+ 装饰线
- 左右双卡片布局（各6"宽），卡片背景 #24243E
- 底部可加总结引用区

### 三段/四段并列页（理论框架类）
- 等宽卡片横向排列
- 每段顶部彩色条（青→金交替）
- 箭头连接表示逻辑递进

### 实施路径页
- 三阶段横向排列，每阶段独立卡片
- 时间标签置于底部
- 颜色编码：青→金→绿

### 总结展望页
- 左右双栏：左"研究总结"（金色标题），右"未来展望"（青色标题）
- 底部引用语（灰色14pt居中）

### 致谢页
- "谢谢！"大字（48pt白色加粗居中）
- 装饰线 + 单位信息

## 核心代码模板
```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN

prs = Presentation()
prs.slide_width = Inches(13.333)
prs.slide_height = Inches(7.5)

def add_bg(slide):
    fill = slide.background.fill; fill.solid(); fill.fore_color.rgb = RGBColor(0x1A, 0x1A, 0x2E)

def add_text_box(slide, left, top, width, height, text, font_size=18, bold=False, color=RGBColor(0xFF,0xFF,0xFF), alignment=PP_ALIGN.LEFT):
    txBox = slide.shapes.add_textbox(left, top, width, height)
    tf = txBox.text_frame; tf.word_wrap = True
    p = tf.paragraphs[0]; p.alignment = alignment
    run = p.add_run(); run.text = text; run.font.size = Pt(font_size); run.font.bold = bold; run.font.color.rgb = color
    return txBox

def add_card(slide, left, top, width, height):
    shape = slide.shapes.add_shape(1, left, top, width, height)
    shape.fill.solid(); shape.fill.fore_color.rgb = RGBColor(0x24, 0x24, 0x3E); shape.line.fill.background()
    return shape

def add_accent_line(slide, left, top, width):
    shape = slide.shapes.add_shape(1, left, top, width, Pt(3))
    shape.fill.solid(); shape.fill.fore_color.rgb = RGBColor(0x00, 0xD9, 0xFF); shape.line.fill.background()
```

## 使用流程
1. 根据内容逻辑设计幻灯片大纲（章节拆分）
2. 选择对应版式模板（封面/目录/双栏/三段并列/路径/总结/致谢）
3. 填充内容，保持配色和字体规范一致
4. 保存为 .pptx

## 注意事项
- 使用 venv Python: `/Users/sunwenning/.hermes/hermes-agent/venv/bin/python3`
- lxml 必须在 venv 中安装（系统Python的lxml有兼容性问题）
- python-pptx 在 venv 中已预装
