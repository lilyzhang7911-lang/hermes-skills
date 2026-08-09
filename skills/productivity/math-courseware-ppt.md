---
name: math-courseware-ppt
description: 制作数学教材复刻风课件PPT，含公式渲染。用于生成含根号/分数/指数的数学课件。
---

# 数学课件 PPT 生成（教材复刻风）

## 触发场景
- 用户要制作数学知识点课件（初中/小学），附了教材截图或给了知识点文本
- 需要把含 √、分数、指数、带分数的数学内容做成 PPT
- 希望公式清晰、可后续编辑、风格统一

## 核心难题与解法
**难点**：python-pptx 文本框里写中文+LaTeX公式会乱码（mathtext 无CJK字形）；纯图片又不方便改。
**解法（已验证可行）**：
1. 中文正文 → PPT 原生文本框（字体用 PingFang SC / Heiti SC，可编辑）
2. 纯数学式（√、分数、指数）→ `matplotlib` mathtext 渲染成**透明背景 PNG** 嵌入
3. 一行"中文+数学"混排 → 拆片段分别渲染后横向拼合为一张透明 PNG 嵌入
关键：matplotlib 的 mathtext 用 **TeX 语法**，注意替代：`\ge`→`\geq`、`\le`→`\leq`、`\ne`→`\neq`、`\Longleftrightarrow`(合法，但 `\iff` 不合法)、`\dfrac`→`\frac`。`-\!3` 表示负间距合法。

## 路线 B2：存量课件 PNG→OMML 原位手术（2026-08-07 实战）
把已有课件里的公式图片换成可编辑 OMML（v4→v5 案例，37 张图全转）：
1. **先 diff 事实源**：v4 文件与当年生成脚本可能已被手动大改（沪教版 vs 华师大版、卡片网格布局等），绝不能重跑脚本，必须解包 pptx 逐页盘点（`<p:pic>` 位置 + rels rId→imageN.png 映射 + `<a:t>` 文本核对）。
2. **转录**：公式 PNG 拼成带文件名标签的 montage 图（4列网格，pad≥20px），vision 逐个转录 LaTeX；可疑的（宽高比反常）单独放大 4 倍复核。
3. **批量验证**：全部 LaTeX 过一遍 pandoc→pptx→grep oMath，全绿再动手。
4. **原位替换**：`shp.shape_type==13` 找 pic，取 `blip@r:embed`→rels→imageN→LaTeX，在原 left/top/width/height 放 textbox 注入 OMML，删 pic。
5. **字号校准**：matplotlib mathtext 28pt 测自然宽高 → 按嵌入盒高(0.5in×0.92)反推 pt，超宽按宽缩，上限 30pt 下限 15pt。**注意**：v4 老图常把长式横向拉伸塞盒（等效 9pt 严重失真），转 OMML 时改逗号拆两行，不复刻失真。
6. **陷阱**：`str.lstrip("\\ ")` 是字符集剥离会吃掉 `\sqrt` 的反斜杠，必须精确 `startswith("\\ ")` 判断。
7. **验证**：解包数 `<m:oMath` 数量 + 残留 `<p:pic>`=0；LibreOffice 渲染只看布局（公式区空白正常），最终效果以用户 PowerPoint/WPS 实测为准。

## 路线 B：OMML 原生公式（pandoc，2026-08-07 验证）
**选型**：最终在学校电脑 Windows PowerPoint / WPS 放映 → 走 OMML；只在本机看 → 路线 A（PNG）。
**优点**：公式是原生对象，双击可用公式编辑器继续改；分数线/根号横线是 Office 标准教材形状，任意缩放清晰；中文与公式可同段混排。
**依赖**：只需 pandoc（内置 texmath，无需 LaTeX 发行版）。本机装在 `~/.local/bin/pandoc`（官方 pkg 经 `pkgutil --expand` + cpio 解包提取，无需管理员密码）。
**工具**：`scripts/slide_omml.py`
- `latex_to_omml(latex)` → 返回可 append 进 `a:p` 的 `a14:m` 节点
- `add_math_paragraph(tf, [("text","当 "),("math",r"a^2+b^2=c^2")], size_pt=24, align="center")` → 中文+公式混排；字号经段落 `defRPr@sz` 设置，公式跟随
- `python slide_omml.py out.pptx` → 自测页（求根公式/勾股/根号/分数）
**关键陷阱**：
- ⚠️ LibreOffice 完全不渲染 OMML（连 pandoc 原生 pptx 都渲染不出公式），soffice 预览链路对 OMML 无效。但用户实测（2026-08-07）：Mac 上的 WPS/Office 类应用能正常渲染 OMML，可直接交付用户本机预览。XML 结构验证靠比对：注入的 `m:oMath` 与 pandoc 原生输出的数学子树（`m:f` bar 分数线 / `m:rad` / `m:sSup`）逐段一致，唯一差异是 inline（`m:oMath`）vs display（`m:oMathPara`）外层包装——混排场景就该用 inline。
- 最终验收让用户在目标放映软件（学校电脑的 PowerPoint/WPS）打开确认。
- pandoc 对 `\ge`/`\le` 友好（直接支持），无需像 matplotlib 那样换 `\geq`。
- 临时 pptx 提取法：pandoc 只输出完整 pptx，脚本从 `ppt/slides/slide1.xml` 里 xpath 取 `a14:m` 节点深拷贝注入，不要试图手写 OMML。

## 依赖修复（本机 venv 已踩坑）
- venv 里 `lxml` 可能损坏 → `venv/bin/pip install --force-reinstall --no-deps lxml`
- 本仓库 python-pptx 是**定制版**：`_Paragraph` 对象**没有 `paragraph_format` 属性**！
  首行缩进不能用 `p.paragraph_format.first_line_indent`，要直接改 XML：
  ```python
  pPr = p._p.get_or_add_pPr()
  pPr.set("marL", str(int(indent_emu)))
  pPr.set("indent", str(-int(indent_emu)))
  ```
- 中文字体必须同时写 `a:latin`/`a:ea`/`a:cs` 的 typeface，否则 Mac 上 fallback 异常
- 预览：用 `soffice --headless --convert-to pdf` 再 `pymupdf(fitz)` 转 PNG；视觉校验逐页

## 工具文件
- `slide_math.py`：数学式渲染器（render_math / compose_line），返回透明 PNG 路径
- 主脚本：定义调色板（教材蓝 #1E4E8C / #1F5CB3、浅黄 #FFF6CC、红 #C02020）、
  header()/footer()/textbox()/rect()/math_para()/math_only() 等封装

## 标准页面结构（13页完整版，华师大八下《实数》风格）
封面 → 本章导航/目标 → 情境导入 → 概念建构(定义+符号+√0=0) → 易错辨析(双重非负性) →
例1(求值) → 例2(化简) → 例3(规律:×100→×10) → 例4(小数小数点规律) → 例5(综合) →
课堂练习(学生版无答案,黄框) → 练习讲评(答案) → 小结+作业

## 验证清单（交付前必做）
1. soffice 转 PDF→PNG
2. 逐页 vision_analyze 检查：中文无乱码、公式清晰、无重叠/溢出/截断
3. 带分数用中文"1又7/9"（文本框）或公式 `$1\frac{7}{9}$`（图片）——练习页用前者更稳
4. 易错点显式总结（教材常未集中的：双重非负性、√a²=|a|）
5. 练习答案单独成页，便于讲评不剧透

## 输出
保存到 `~/Desktop/<知识点>_课件.pptx`，并转 PNG 预览供用户确认。
