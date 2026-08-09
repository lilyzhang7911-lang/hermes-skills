---
name: security-gate
description: 安全左移门禁。在 commit/部署前强制安全检查，发现安全问题时触发 STOP → 修复 → 旋转密钥 → 全局扫描流程。源自 ECC v2.0.0 security-reviewer agent。
tags: [security, code-review, gate, agent-orchestration]
---

# 安全左移门禁 (Security-First Gate)

## 概述

将安全审查从"事后补救"升级为**commit/部署前的强制拦截**。任何涉及密钥、用户输入、网络请求的代码变更，必须通过 security-gate 才能进入下一环节。

## 触发条件

- 代码涉及 API key / 密码 / token / 证书
- 处理用户输入（表单、URL 参数、文件上传）
- 数据库查询/操作（SQL、ORM）
- HTTP 请求/响应处理
- 认证/授权逻辑
- 任何"敏感数据"相关变更

## Commit 前强制检查项

```markdown
- [ ] 无硬编码密钥/密码/token（使用环境变量或密钥管理器）
- [ ] 所有用户输入已校验（Schema-based validation, fail fast）
- [ ] SQL 注入防护（参数化查询，禁止字符串拼接）
- [ ] XSS 防护（HTML 转义，CSP header）
- [ ] CSRF 保护启用
- [ ] 认证/授权验证通过（未授权请求被拒绝）
- [ ] 所有端点有速率限制
- [ ] 错误信息不泄露敏感数据（堆栈跟踪、内部路径、数据库结构）
```

## 密钥管理规则

```
❌ 硬编码在代码中（即使注释标记为"临时"）
✅ 环境变量或密钥管理器（Vault/1Password/Secrets Manager）
✅ 启动时校验必需密钥存在，缺失则拒绝启动
✅ 暴露后立即旋转（rotate），并全局扫描同类问题
```

## STOP 响应链（发现安全问题时）

```
STOP → 停止当前所有操作
    ↓
security-reviewer agent 介入分析
    ↓
修复 CRITICAL 级别问题（阻塞性漏洞）
    ↓
旋转已暴露的密钥/凭证
    ↓
全局扫描代码库中同类模式（如其他硬编码密钥、未参数化的 SQL）
    ↓
确认全部修复后，方可继续
```

## Agent 编排映射

| 场景 | 触发的 Agent | 优先级 |
|------|-------------|--------|
| Commit 前安全检查 | security-reviewer | **强制拦截**（STOP） |
| 发现硬编码密钥 | security-reviewer + 人工确认 | CRITICAL |
| SQL 注入风险 | security-reviewer → build-error-resolver | HIGH |
| XSS/CSRF 漏洞 | security-reviewer → code-reviewer (并行) | HIGH |

## 荷妹主动触发规则

```
检测到敏感代码变更
    ↓
自动触发 security-gate 检查
    ├─ 通过 → 继续流程（commit/deploy）
    └─ 未通过 → STOP → security-reviewer 介入
        ↓
修复 → 旋转密钥 → 全局扫描 → 重新检查
```

## 与文宁哲学的映射

安全左移体现了"到位而不越位"的边界感：
- **到位** = 在关键节点设置强制门禁，不放过任何安全隐患
- **不越位** = 不替代开发者的判断，而是提供检查清单和自动扫描

---

*参考：ECC AGENTS.md "Security Guidelines", skills/security-review/*
