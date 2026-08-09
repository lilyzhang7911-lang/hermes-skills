---
name: github-skill-tools
description: "通过 OmniRoute MCP 工具链搜索、扫描和安装 GitHub 上的 Agent Skills（SKILL.md）。支持按 stars/score 过滤，恶意内容扫描，多目标安装。"
category: productivity
version: 1.0.0
---

# GitHub Skill Tools (MCP)

## Trigger Conditions
- 用户想从 GitHub 发现/搜索可用的 Agent Skills
- 需要评估某个 skill repo 的安全性（malware/secrets 扫描）
- 需要将发现的 skills 安装到 Hermes/Claude/Gemini/OpenCode

## Available Tools (3 MCP tools)

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `github_skills_search` | 搜索 GitHub 上含 SKILL.md 的仓库 | `query`, `minStars`, `maxResults`, `minScore` |
| `github_skills_scan` | 扫描 skill 内容为安全威胁（malware, secrets） | `content`(文本), `repoName` |
| `github_skills_install` | 将 skill 安装到指定目标 | `repoName`, `targets` (hermes/claude/gemini/opencode) |

## Workflow

### 1. 搜索 Skills
```
github_skills_search: query="llm", minStars=100, maxResults=20, minScore=70
```
返回：skills 列表（fullName, stars, score, description, topics, htmlUrl）+ errors

### 2. 安全扫描
```
github_skills_scan: content="<skill.md内容>", repoName="owner/repo"
```
返回：clean (bool) + findings[]（pattern + context）

### 3. 安装到 Hermes
```
github_skills_install: repoName="owner/repo", targets=["hermes"]
```
返回：每个 target 的 install 结果（ok, destDir, error）

## Pitfalls
- `minScore` 过滤是可选的，设为 0 不过滤
- scan 只检查文本内容中的 blocked patterns（malware signatures, exposed secrets）
- install 目前是 dry-run 模式，返回计划路径但不实际克隆
- targets 必须是 INSTALL_TARGETS 之一：hermes / claude / gemini / opencode

## Verification
- search 返回的 skills 应包含 score >= minScore 的条目
- scan 的 clean=true 表示无安全威胁
- install 结果中 ok=true 表示安装成功，destDir 为实际目标目录
