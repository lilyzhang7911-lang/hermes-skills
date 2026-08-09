---
name: markitdown
description: "Convert various files to Markdown via MarkItDown (Microsoft). Supports PDF, DOCX, PPTX, XLSX, Images, Audio, YouTube, EPUB."
rational: true
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [PDF, DOCX, PPTX, XLSX, Audio, YouTube, EPUB, OCR, Conversion, Microsoft]
    related_skills: [ocr-and-documents, nano-pdf, powerpoint]
---

# MarkItDown Integration

MarkItDown is a lightweight Python utility from the Microsoft AutoGen Team for converting various files to Markdown. It serves as a **unified conversion gateway** — complementing our existing `ocr-and-documents` skill by adding YouTube transcription, audio transcription, EPUB support, and LLM-Vision OCR capabilities.

## Installation

### Prerequisites
- **Python 3.10+** (系统默认 Python 3.9 不兼容，需使用 `/Users/sunwenning/.local/bin/python3.11` 或 `/opt/homebrew/bin/python3.12`)
- macOS 下 pip 默认受 externally-managed-environment 保护，必须通过虚拟环境安装

### 国内镜像安装（推荐）

```bash
# 1. 创建虚拟环境
cd /Users/sunwenning/Desktop/MyWorkHome
/Users/sunwenning/.local/bin/python3.11 -m venv markitdown-venv

# 2. 激活虚拟环境
source ./markitdown-venv/bin/activate

# 3. 使用清华镜像安装（解决翻墙困难）
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ 'markitdown[all]'

# 4. 验证安装
python -c "import markitdown; print(f'MarkItDown version: {markitdown.__version__}')"
```

### 国内镜像源备选

| 镜像站 | URL |
|--------|-----|
| 清华大学 | https://pypi.tuna.tsinghua.edu.cn/simple/ |
| 阿里云 | https://mirrors.aliyun.com/pypi/simple/ |
| 华为云 | https://repo.huaweicloud.com/repository/pypi/simple/ |

### 选择性安装（减小体积）

```bash
# 仅 PDF + Word + PPTX
pip install 'markitdown[pdf, docx, pptx]'

# YouTube 转录
pip install 'markitdown[youtube-transcription]'

# Audio 转录
pip install 'markitdown[audio-transcription]'
```

### OCR 增强插件（可选）

```bash
# 安装 markitdown-ocr 插件
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ markitdown-ocr

# 验证
python -c "import markitdown_ocr; print('OCR plugin loaded')"
```

## Usage Patterns

### CLI — Quick Conversion

```bash
# Convert a file and redirect output
markitdown path-to-file.pdf > document.md

# Specify output file
markitdown path-to-file.pdf -o document.md

# Pipe content
cat path-to-file.pdf | markitdown
```

### Python API — Programmatic Use

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=False)  # Set True to enable plugins
result = md.convert("test.xlsx")
print(result.text_content)
```

### LLM-Vision OCR (for scanned documents with embedded images)

```python
from markitdown import MarkItDown
from openai import OpenAI

# Use any OpenAI-compatible client (e.g., LM Studio)
client = OpenAI()  # or OpenAI(base_url="http://localhost:1234/v1") for LM Studio
md = MarkItDown(llm_client=client, llm_model="gpt-4o", llm_prompt="optional custom prompt")
result = md.convert("example.jpg")
print(result.text_content)
```

### OCR Plugin (markitdown-ocr) — Enhanced PDF/DOCX/PPTX/XLSX Conversion

The `markitdown-ocr` plugin adds LLM Vision-based OCR for embedded images in documents:

```python
from markitdown import MarkItDown
from openai import OpenAI

md = MarkItDown(
    enable_plugins=True,
    llm_client=OpenAI(),
    llm_model="gpt-4o",
)
result = md.convert("document_with_images.pdf")
print(result.text_content)
```

**Note**: If no `llm_client` is provided, the OCR plugin loads but silently falls back to built-in converters.

### Azure Content Understanding (Cloud-powered conversion)

For higher-quality extraction with structured field extraction:

```python
from markitdown import MarkItDown

# Zero-config — auto-selects analyzer per file type
md = MarkItDown(cu_endpoint="<content_understanding_endpoint>")
result = md.convert("report.pdf")   # documents → prebuilt-documentSearch
result = md.convert("meeting.mp4")  # video → prebuilt-videoSearch
result = md.convert("call.wav")     # audio → prebuilt-audioSearch
print(result.markdown)

# With custom analyzer for domain-specific field extraction
md = MarkItDown(
    cu_endpoint="<content_understanding_endpoint>",
    cu_analyzer_id="my-invoice-analyzer",
)
result = md.convert("invoice.pdf")
print(result.markdown)
```

### Azure Document Intelligence

```bash
markitdown path-to-file.pdf -o document.md -d -e "<document_intelligence_endpoint>"
```

## Plugin System

MarkItDown supports 3rd-party plugins (disabled by default):

```bash
# List installed plugins
markitdown --list-plugins

# Enable plugins for a conversion
markitdown --use-plugins path-to-file.pdf
```

To find available plugins, search GitHub for `#markitdown-plugin`. See `packages/markitdown-sample-plugin` for plugin development guidance.

## Supported Formats & Optional Dependencies

| Format | Dependency Tag | Notes |
|--------|---------------|-------|
| PDF | `[pdf]` | Built-in + OCR plugin + Azure options |
| PowerPoint | `[pptx]` | Built-in + OCR plugin |
| Word | `[docx]` | Built-in + OCR plugin |
| Excel | `[xlsx]` / `[xls]` | Built-in + OCR plugin |
| Images | — | EXIF metadata + LLM Vision descriptions |
| Audio | `[audio-transcription]` | WAV/MP3 speech transcription |
| YouTube | `[youtube-transcription]` | Video transcription |
| HTML | — | Native support |
| CSV/JSON/XML | — | Native support |
| ZIP | — | Iterates over contents |
| EPUB | — | Native support |

## When to Use MarkItDown vs Other Skills

| Scenario | Recommended Tool | Reason |
|----------|------------------|--------|
| Text-based PDF extraction (fast) | `ocr-and-documents` (pymupdf) | Instant, no models needed |
| Scanned PDF with images + LLM OCR | MarkItDown + `markitdown-ocr` | LLM Vision for image text |
| YouTube video transcription | MarkItDown (`[youtube-transcription]`) | Exclusive capability |
| Audio file transcription | MarkItDown (`[audio-transcription]`) | Exclusive capability |
| Word/Excel/PPTX conversion | MarkItDown or `ocr-and-documents` | Both work; MarkItDown has broader format support |
| PDF text editing (typos/titles) | `nano-pdf` | NL-based editing, not extraction |
| Quick HTML/CSV/JSON conversion | MarkItDown | Native support, no extra deps |

## Security Considerations

MarkItDown performs I/O with the privileges of the current process. Like `open()` or `requests.get()`, it will access resources that the process itself can access.

- **Sanitize inputs**: Do not pass untrusted input directly to MarkItDown
- **Use narrowest API**: Prefer `convert_local()` for local files, `convert_stream()` for streams, rather than the permissive `convert()`
- In hosted/server-side applications, validate and restrict file paths, URI schemes, and network destinations before calling MarkItDown

## Docker Usage

```bash
docker build -t markitdown:latest .
docker run --rm -i markitdown:latest < ~/your-file.pdf > output.md
```

## MCP Server Integration

The `markitdown-mcp` package allows running MarkItDown as an MCP server:

```bash
# Run as MCP server (port configurable)
markitdown-mcp --port 8080
```

This enables Hermes to call MarkItDown conversion via MCP protocol — useful for integrating document conversion into agent workflows.

## Notes

- Output is Markdown optimized for LLM consumption (headings, lists, tables, links preserved)
- Not intended for high-fidelity human-readable conversions
- Plugin converters use priority-based registration (-1.0 for OCR-enhanced, 0.0 for built-in)
- For Word docs: `python-docx` parses actual structure (better than OCR)
- For PowerPoint: see the `powerpoint` skill for full slide/notes support
