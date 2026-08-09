---
name: lsp-integration
description: LSP 集成客户端 — 代码诊断、符号跳转、批量重构。从 Grok Build xai-grok-tools LSP 工程化实践提取，支持 Python/Rust/Shell/JS 多语言。
version: 1.0.0
tags: [lsp, diagnostics, code-review, refactoring, multi-language]
category: engineering
---

# LSP 集成技能

## 触发条件
当用户需要：
- 对代码文件运行诊断（lint/check）
- 查找符号定义/调用者/引用
- 批量重命名跨文件的符号
- 检测未使用的导入
- 快速了解项目代码质量

## 核心实现

### 1. 主脚本：lsp_client.sh

**路径：** `~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh`

#### 命令一览

| 命令 | 用法 | 功能 |
|------|------|------|
| `diagnose` | `<file> [python\|rust\|shell\|auto]` | 代码诊断（ruff→flake8→pylint / rustc/cargo / shellcheck） |
| `goto` | `<symbol> [file]` | 符号定位跳转（ripgrep全文搜索） |
| `callers` | `<symbol>` | 查找函数调用者 |
| `references` | `<symbol>` | 引用统计 |
| `rename` | `<old_name> <new_name> [dry-run]` | 批量重命名（默认dry-run安全模式） |
| `unused-imports` | `<file.py>` | Python未使用导入检测 |

#### 诊断优先级链

```
Python: ruff → flake8 → pylint（自动降级，最快优先）
Rust:   rustc --parse-only → cargo check（单文件/项目模式）
Shell:  shellcheck（如有安装）
JS/TS:  ripgrep全文搜索（goto/callers/references）
```

### 2. 使用示例

```bash
# Python诊断
~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh diagnose src/main.py auto

# Rust诊断
~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh diagnose src/lib.rs rust

# 查找符号定义
~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh goto "Hegelian Dialectic"

# 查找函数调用者
~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh callers "web_fetch"

# 引用统计
~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh references "memory_index"

# 批量重命名（dry-run）
~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh rename "old_func" "new_func" true

# 实际执行重命名
~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh rename "old_func" "new_func" false

# Python未使用导入
~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh unused-imports src/main.py
```

### 3. Hermes Agent 集成

通过 `terminal` 工具调用：

```python
from hermes_tools import terminal

# 诊断代码
result = terminal(
    command="~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh diagnose src/main.py auto",
    timeout=30
)

# 查找符号
result = terminal(
    command="~/.hermes/skills/engineering/lsp-integration/scripts/lsp_client.sh goto 'Hegelian Dialectic'",
    timeout=15
)
```

## 依赖要求

| 工具 | 必需/可选 | 安装方式 |
|------|-----------|----------|
| ripgrep (rg) | ✅ 必需 | `brew install ripgrep`（macOS通常已预装） |
| Python 3.9+ | ✅ 必需 | macOS自带或 `brew install python` |
| SQLite3 | ✅ 必需 | macOS自带 |
| ruff | ⭐ 推荐 | `pip install ruff`（最快Python linter） |
| shellcheck | ⭐ 推荐 | `brew install shellcheck` |
| flake8 | 回退 | `pip install flake8` |
| pylint | 回退 | `pip install pylint` |

## 注意事项

1. **安全优先**：rename命令默认dry-run，需显式传`false`才执行实际修改
2. **自动降级**：diagnose按 ruff→flake8→pylint 优先级链自动选择可用工具
3. **ripgrep搜索范围**：自动排除 .git、node_modules、.Trash 目录
4. **Python AST分析**：unused-imports使用标准库ast模块，无需额外依赖

## 扩展点

- 可集成 ESLint/Prettier 支持 JS/TS 诊断
- 可添加 Go/VimL/C++ 语言支持
- 可与 Hermes Agent compaction 联动，在上下文压缩前自动扫描代码质量
- 可扩展为 CI pre-commit hook（git hooks）
