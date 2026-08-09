---
name: code-review
description: "代码审查统一入口：通用审查、GitHub PR 审查、Pre-commit 验证流水线。"
tags: [code-review, github, pr, security, verification, quality]
---

# 代码审查

统一入口：覆盖通用代码审查、GitHub PR 审查、Pre-commit 验证流水线。

## 场景决策树

```
开始
├─ 审查自己的代码（提交前）？ → Pre-commit 验证流水线
├─ 审查别人的 PR（GitHub）？ → GitHub PR 审查
├─ 通用代码审查（标准+规格）？ → 通用审查流程
└─ 审查过度工程？ → ponytail skill
```

## 一、Pre-commit 验证流水线

**适用**：提交/推送前验证自己的代码  
**触发**：用户说 "commit"、"push"、"ship"、"verify"、"review before merge"  
**核心原则**：No agent should verify its own work. Fresh context finds what you miss.

### Step 1 — 获取 diff

```bash
git diff --cached  # 暂存的更改
# 如果为空，尝试 git diff 或 git diff HEAD~1 HEAD
```

如果 diff 超过 15,000 字符，按文件拆分：
```bash
git diff --name-only
git diff HEAD -- specific_file.py
```

### Step 2 — 静态安全扫描

只扫描新增行：

```bash
# 硬编码密钥
git diff --cached | grep "^+" | grep -iE "(api_key|secret|password|token|passwd)\s*=\s*['\"][^'\"]{6,}['\"]"

# Shell 注入
git diff --cached | grep "^+" | grep -E "os\.system\(|subprocess.*shell=True"

# 危险的 eval/exec
git diff --cached | grep "^+" | grep -E "\beval\(|\bexec\("

# 不安全的反序列化
git diff --cached | grep "^+" | grep -E "pickle\.loads?\("

# SQL 注入
git diff --cached | grep "^+" | grep -E "execute\(f\"|\.format\(.*SELECT|\.format\(.*INSERT"
```

### Step 3 — 基线测试和 Linting

检测项目语言并运行相应工具。**关键**：先捕获基线失败数（stash → run → pop），只计算你的更改引入的新失败。

```bash
# Python
python -m pytest --tb=no -q 2>&1 | tail -5
which ruff && ruff check . 2>&1 | tail -10
which mypy && mypy . --ignore-missing-imports 2>&1 | tail -10

# Node
npm test -- --passWithNoTests 2>&1 | tail -5
which npx && npx eslint . 2>&1 | tail -10
which npx && npx tsc --noEmit 2>&1 | tail -10

# Rust
cargo test 2>&1 | tail -5
cargo clippy -- -D warnings 2>&1 | tail -10

# Go
go test ./... 2>&1 | tail -5
which go && go vet ./... 2>&1 | tail -10
```

### Step 4 — 自审清单

- [ ] 无硬编码密钥、API keys、凭据
- [ ] 用户输入有验证
- [ ] SQL 查询使用参数化语句
- [ ] 文件操作验证路径（无遍历）
- [ ] 外部调用有错误处理
- [ ] 无遗留的 debug print/console.log
- [ ] 无注释掉的代码
- [ ] 新代码有测试（如果有测试套件）

### Step 5 — 独立审查者子代理

调用 `delegate_task`，审查者只获得 diff 和静态扫描结果，与实现者无共享上下文。

```python
delegate_task(
    goal="""You are an independent code reviewer. Review the git diff and return ONLY valid JSON.

FAIL-CLOSED RULES:
- security_concerns non-empty -> passed must be false
- logic_errors non-empty -> passed must be false
- Cannot parse diff -> passed must be false
- Only set passed=true when BOTH lists are empty

SECURITY (auto-FAIL): hardcoded secrets, backdoors, data exfiltration,
shell injection, SQL injection, path traversal, eval()/exec() with user input,
pickle.loads(), obfuscated commands.

LOGIC ERRORS (auto-FAIL): wrong conditional logic, missing error handling for
I/O/network/DB, off-by-one errors, race conditions, code contradicts intent.

SUGGESTIONS (non-blocking): missing tests, style, performance, naming.

<static_scan_results>
[INSERT FINDINGS FROM STEP 2]
</static_scan_results>

<code_changes>
---
[INSERT GIT DIFF OUTPUT]
---
</code_changes>

Return ONLY this JSON:
{
  "passed": true or false,
  "security_concerns": [],
  "logic_errors": [],
  "suggestions": [],
  "summary": "one sentence verdict"
}""",
    context="Independent code review. Return only JSON verdict.",
    toolsets=["terminal"]
)
```

### Step 6 — 评估结果

**全部通过**：→ Step 8（提交）  
**任何失败**：→ Step 7（自动修复）

### Step 7 — 自动修复循环

**最多 2 次修复-重验证循环**。生成第三个代理上下文（不是实现者，不是审查者）：

```python
delegate_task(
    goal="""You are a code fix agent. Fix ONLY the specific issues listed below.
Do NOT refactor, rename, or change anything else.

Issues to fix:
---
[INSERT security_concerns AND logic_errors FROM REVIEWER]
---

Current diff for context:
---
[INSERT GIT DIFF]
---

Fix each issue precisely. Describe what you changed and why.""",
    context="Fix only the reported issues. Do not change anything else.",
    toolsets=["terminal", "file"]
)
```

修复后重新运行 Steps 1-6。2 次失败后升级给用户。

### Step 8 — 提交

```bash
git add -A && git commit -m "[verified] <description>"
```

## 二、GitHub PR 审查

**适用**：审查 GitHub 上的 PR，留内联评论  
**前提**：已认证 GitHub（见 github-auth skill）

### 环境设置

```bash
# 检测认证方式
if command -v gh &>/dev/null && gh auth status &>/dev/null; then
  AUTH="gh"
else
  AUTH="git"
  # 从 ~/.git-credentials 或 HERMES_HOME/.env 获取 GITHUB_TOKEN
fi

REMOTE_URL=$(git remote get-url origin)
OWNER_REPO=$(echo "$REMOTE_URL" | sed -E 's|.*github\.com[:/]||; s|\.git$||')
OWNER=$(echo "$OWNER_REPO" | cut -d/ -f1)
REPO=$(echo "$OWNER_REPO" | cut -d/ -f2)
```

### 查看 PR 详情

**With gh:**
```bash
gh pr view 123
gh pr diff 123
gh pr diff 123 --name-only
gh pr checks 123
```

**With curl:**
```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/pulls/$PR_NUMBER \
  | python3 -c "import sys, json; pr = json.load(sys.stdin); print(f'Title: {pr[\"title\"]}')"
```

### 本地检出 PR

```bash
git fetch origin pull/123/head:pr-123
git checkout pr-123
# 现在可以用 read_file, search_files, 运行测试等
```

### 审查清单

#### 正确性
- 代码是否做了它声称的事？
- 边界情况处理（空输入、null、大数据、并发）？
- 错误路径优雅处理？

#### 安全性
- 无硬编码密钥、凭据、API keys
- 用户输入有验证
- 无 SQL 注入、XSS、路径遍历
- 需要时有 auth/authz 检查

#### 代码质量
- 命名清晰（变量、函数、类）
- 无不必要的复杂性或过早抽象
- DRY — 无应提取的重复逻辑
- 函数专注（单一职责）

#### 测试
- 新代码路径有测试？
- 快乐路径和错误情况都覆盖？
- 测试可读且可维护？

#### 性能
- 无 N+1 查询或不必要的循环
- 适当使用缓存
- 异步代码路径无阻塞操作

#### 文档
- 公共 API 有文档
- 非显而易见的逻辑有注释解释"为什么"
- 行为变化时 README 已更新

### 留内联评论

**With gh:**
```bash
HEAD_SHA=$(gh pr view 123 --json headRefOid --jq '.headRefOid')
gh api repos/$OWNER/$REPO/pulls/123/comments \
  --method POST \
  -f body="This could be simplified with a list comprehension." \
  -f path="src/auth/login.py" \
  -f commit_id="$HEAD_SHA" \
  -f line=45 \
  -f side="RIGHT"
```

**With curl:**
```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/pulls/$PR_NUMBER/comments \
  -d "{
    \"body\": \"This could be simplified.\",
    \"path\": \"src/auth/login.py\",
    \"commit_id\": \"$HEAD_SHA\",
    \"line\": 45,
    \"side\": \"RIGHT\"
  }"
```

### 提交正式审查

```bash
# 批准
gh pr review 123 --approve --body "LGTM!"

# 请求更改
gh pr review 123 --request-changes --body "See inline comments."

# 仅评论
gh pr review 123 --comment --body "Some suggestions, nothing blocking."
```

### 审查输出格式

```markdown
## Code Review Summary

### 🔴 Critical
- **src/auth.py:45** — SQL injection: user input passed directly to query.
  Suggestion: Use parameterized queries.

### ⚠️ Warnings
- **src/models/user.py:23** — Password stored in plaintext. Use bcrypt.
- **src/api/routes.py:112** — No rate limiting on login endpoint.

### 💡 Suggestions
- **src/utils/helpers.py:8** — Duplicates logic in core/utils.py:34.
- **tests/test_auth.py** — Missing edge case: expired token test.

### ✅ Looks Good
- Clean separation of concerns in the middleware layer
- Good test coverage for the happy path
```

### 决策：Approve vs Request Changes vs Comment

- **Approve** — 无 critical/warning 级别问题
- **Request Changes** — 有 critical/warning 级别问题需修复
- **Comment** — 观察和建议，无阻塞（用于不确定或 draft PR）

### 清理

```bash
git checkout main
git branch -D pr-$PR_NUMBER
```

## 三、通用审查流程

**适用**：审查自某个固定点（commit、branch、tag）以来的更改  
**两个维度**：Standards（代码是否遵循仓库的编码标准？）+ Spec（代码是否匹配原始 issue/PRD 的要求？）

### 获取差异

```bash
# 暂存的更改
git diff --staged

# 与 main 的所有更改
git diff main...HEAD

# 仅文件名
git diff main...HEAD --name-only

# 统计摘要
git diff main...HEAD --stat
```

### 审查策略

1. **先看大局**：
```bash
git diff main...HEAD --stat
git log main..HEAD --oneline
```

2. **逐文件审查** — 用 `read_file` 获取完整上下文：
```bash
git diff main...HEAD -- src/auth/login.py
```

3. **检查常见问题**：
```bash
# 遗留的调试语句
git diff main...HEAD | grep -n "print(\|console\.log\|TODO\|FIXME\|HACK\|XXX\|debugger"

# 意外暂存的大文件
git diff main...HEAD --stat | sort -t'|' -k2 -rn | head -10

# 密钥或凭据模式
git diff main...HEAD | grep -in "password\|secret\|api_key\|token.*=\|private_key"

# 合并冲突标记
git diff main...HEAD | grep -n "<<<<<<\|>>>>>>\|======="
```

## 四、常见陷阱汇总

### Pre-commit 流水线
- **空 diff** — 检查 `git status`，告诉用户没有可验证的内容
- **不是 git 仓库** — 跳过并告知用户
- **大 diff（>15k 字符）** — 按文件拆分，分别审查
- **delegate_task 返回非 JSON** — 重试一次，然后视为 FAIL
- **误报** — 如果审查者标记了有意为之的内容，在 fix prompt 中注明
- **无测试框架** — 跳过回归检查，审查者判定仍然运行
- **Lint 工具未安装** — 静默跳过，不失败

### GitHub PR 审查
- **未认证** — 先运行 github-auth skill
- **line 字段** — 指的是新版本的行号。删除的行用 `"side": "LEFT"`
- **并发编辑** — 提交审查前确保没有其他审查在进行

### 通用
- **不要自己验证自己的代码** — 用 delegate_task 获取独立审查
- **基线比较** — 只计算你的更改引入的新失败，不是所有失败
- **自动修复最多 2 次** — 之后升级给用户

## 五、参考资源

- GitHub 认证：`github-auth` skill
- TDD 工作流：`tdd` skill
- 过度工程审查：`ponytail` skill
