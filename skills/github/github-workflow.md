---
name: github-workflow
description: "GitHub 工作流统一入口：认证、仓库管理、Issue、PR、代码分析。"
tags: [github, workflow, auth, issues, pr, repository]
---

# GitHub 工作流

统一入口：覆盖认证、仓库管理、Issue 管理、PR 工作流、代码分析。

## 场景决策树

```
开始
├─ 需要设置 GitHub 认证？ → 认证设置
├─ 需要克隆/创建/fork 仓库？ → 仓库管理
├─ 需要管理 Issues？ → Issue 管理
├─ 需要创建/审查/合并 PR？ → PR 工作流
└─ 需要分析代码库（LOC/语言）？ → 代码分析
```

## 一、认证设置

### 检测当前认证状态

```bash
# 检查可用工具
git --version
gh --version 2>/dev/null || echo "gh not installed"

# 检查认证状态
gh auth status 2>/dev/null || echo "gh not authenticated"
git config --global credential.helper 2>/dev/null || echo "no git credential helper"
```

### 决策树

1. 如果 `gh auth status` 显示已认证 → 使用 `gh` 完成所有操作
2. 如果 `gh` 已安装但未认证 → 使用 "gh auth" 方法
3. 如果 `gh` 未安装 → 使用 "git-only" 方法（无需 sudo）

### 方法 1：仅 Git 认证（无 gh，无 sudo）

#### 选项 A：HTTPS + 个人访问令牌（推荐）

**步骤 1：创建个人访问令牌**

告诉用户访问：**https://github.com/settings/tokens**

- 点击 "Generate new token (classic)"
- 命名如 "hermes-agent"
- 选择范围：
  - `repo`（完整仓库访问 — 读、写、推送、PR）
  - `workflow`（触发和管理 GitHub Actions）
  - `read:org`（如果使用组织仓库）
- 设置过期时间（90 天是好的默认值）
- 复制令牌 — 不会再显示

**步骤 2：配置 git 存储令牌**

```bash
# 设置凭据助手缓存凭据
# "store" 保存到 ~/.git-credentials（明文，简单，持久）
git config --global credential.helper store

# 现在执行触发认证的测试操作 — git 会提示输入凭据
# 用户名：<their-github-username>
# 密码：<粘贴个人访问令牌，不是 GitHub 密码>
git ls-remote https://github.com/<their-username>/<any-repo>.git
```

输入凭据一次后，它们被保存并用于所有未来操作。

**替代方案：缓存助手（凭据从内存过期）**

```bash
# 在内存中缓存 8 小时（28800 秒）而不是保存到磁盘
git config --global credential.helper 'cache --timeout=28800'
```

**替代方案：直接在远程 URL 中设置令牌（每仓库）**

```bash
# 在远程 URL 中嵌入令牌（完全避免凭据提示）
git remote set-url origin https://<username>:<token>@github.com/<owner>/<repo>.git
```

**步骤 3：配置 git 身份**

```bash
# 提交必需 — 设置姓名和邮箱
git config --global user.name "Their Name"
git config --global user.email "their-email@example.com"
```

**步骤 4：验证**

```bash
# 测试推送访问（现在应该无需任何提示工作）
git ls-remote https://github.com/<their-username>/<any-repo>.git

# 验证身份
git config --global user.name
git config --global user.email
```

#### 选项 B：SSH 密钥认证

适合偏好 SSH 或已有密钥设置的用户。

**步骤 1：检查现有 SSH 密钥**

```bash
ls -la ~/.ssh/id_*.pub 2>/dev/null || echo "No SSH keys found"
```

**步骤 2：如需要生成密钥**

```bash
# 生成 ed25519 密钥（现代、安全、快速）
ssh-keygen -t ed25519 -C "their-email@example.com" -f ~/.ssh/id_ed25519 -N ""

# 显示公钥供用户添加到 GitHub
cat ~/.ssh/id_ed25519.pub
```

告诉用户在 **https://github.com/settings/keys** 添加公钥
- 点击 "New SSH key"
- 粘贴公钥内容
- 命名如 "hermes-agent-<machine-name>"

**步骤 3：测试连接**

```bash
ssh -T git@github.com
# 预期："Hi <username>! You've successfully authenticated..."
```

**步骤 4：配置 git 对 GitHub 使用 SSH**

```bash
# 自动将 HTTPS GitHub URL 重写为 SSH
git config --global url."git@github.com:".insteadOf "https://github.com/"
```

**步骤 5：配置 git 身份**

```bash
git config --global user.name "Their Name"
git config --global user.email "their-email@example.com"
```

### 方法 2：gh CLI 认证

如果 `gh` 已安装，它在一个步骤中处理 API 访问和 git 凭据。

#### 交互式浏览器登录（桌面）

```bash
gh auth login
# 选择：GitHub.com
# 选择：HTTPS
# 通过浏览器认证
```

#### 基于令牌的登录（无头/SSH 服务器）

```bash
echo "<THEIR_TOKEN>" | gh auth login --with-token

# 通过 gh 设置 git 凭据
gh auth setup-git
```

#### 验证

```bash
gh auth status
```

### 无 gh 时使用 GitHub API

当 `gh` 不可用时，仍可以使用 `curl` 和个人访问令牌访问完整 GitHub API。

#### 设置 API 调用令牌

```bash
# 选项 1：导出为环境变量（推荐 — 保持在命令外）
export GITHUB_TOKEN="<token>"

# 然后在 curl 调用中使用：
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user
```

#### 从 Git 凭据提取令牌

如果 git 凭据已配置（通过 credential.helper store），可以提取：

```bash
# 从 git 凭据存储读取
uv run python3 "${HERMES_HOME:-$HOME/.hermes}/skills/github/github-auth/scripts/git-credential-token.py"
```

#### 助手：检测认证方法

在任何 GitHub 工作流开始时使用此模式：

```bash
# 先尝试 gh，回退到 git + curl
if command -v gh &>/dev/null && gh auth status &>/dev/null; then
  echo "AUTH_METHOD=gh"
elif [ -n "$GITHUB_TOKEN" ]; then
  echo "AUTH_METHOD=curl"
elif _hermes_env="${HERMES_HOME:-$HOME/.hermes}/.env"; [ -f "$_hermes_env" ] && grep -q "^GITHUB_TOKEN=" "$_hermes_env"; then
  export GITHUB_TOKEN=$(grep "^GITHUB_TOKEN=" "$_hermes_env" | head -1 | cut -d= -f2 | tr -d '\n\r')
  echo "AUTH_METHOD=curl"
elif grep -q "github.com" ~/.git-credentials 2>/dev/null; then
  export GITHUB_TOKEN=$(uv run python3 "${HERMES_HOME:-$HOME/.hermes}/skills/github/github-auth/scripts/git-credential-token.py")
  echo "AUTH_METHOD=curl"
else
  echo "AUTH_METHOD=none"
  echo "Need to set up authentication first"
fi
```

### 故障排除

| 问题 | 解决方案 |
|------|---------|
| `git push` 要求密码 | GitHub 禁用了密码认证。使用个人访问令牌作为密码，或切换到 SSH |
| `remote: Permission to X denied` | 令牌可能缺少 `repo` 范围 — 用正确范围重新生成 |
| `fatal: Authentication failed` | 缓存凭据可能过期 — 运行 `git credential reject` 然后重新认证 |
| `ssh: connect to host github.com port 22: Connection refused` | 尝试通过 HTTPS 端口 443 的 SSH：在 `~/.ssh/config` 中添加 `Host github.com` 与 `Port 443` 和 `Hostname ssh.github.com` |
| 凭据不持久 | 检查 `git config --global credential.helper` — 必须是 `store` 或 `cache` |
| 多个 GitHub 账户 | 在 `~/.ssh/config` 中使用 SSH 与每个主机别名的不同密钥，或每仓库凭据 URL |
| `gh: command not found` + 无 sudo | 使用上面的 git-only 方法 1 — 无需安装 |

## 二、仓库管理

### 克隆仓库

克隆是纯 `git` — 两种方式相同：

```bash
# 通过 HTTPS 克隆（与凭据助手或令牌嵌入 URL 一起工作）
git clone https://github.com/owner/repo-name.git

# 克隆到特定目录
git clone https://github.com/owner/repo-name.git ./my-local-dir

# 浅克隆（对大仓库更快）
git clone --depth 1 https://github.com/owner/repo-name.git

# 克隆特定分支
git clone --branch develop https://github.com/owner/repo-name.git

# 通过 SSH 克隆（如果配置了 SSH）
git clone git@github.com:owner/repo-name.git
```

**使用 gh（简写）：**

```bash
gh repo clone owner/repo-name
gh repo clone owner/repo-name -- --depth 1
```

### 创建仓库

**使用 gh：**

```bash
# 创建公共仓库并克隆
gh repo create my-new-project --public --clone

# 私有，带描述和许可证
gh repo create my-new-project --private --description "A useful tool" --license MIT --clone

# 在组织下
gh repo create my-org/my-new-project --public --clone

# 从现有本地目录
cd /path/to/existing/project
gh repo create my-project --source . --public --push
```

**使用 git + curl：**

```bash
# 通过 API 创建远程仓库
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/user/repos \
  -d '{
    "name": "my-new-project",
    "description": "A useful tool",
    "private": false,
    "auto_init": true,
    "license_template": "mit"
  }'

# 克隆它
git clone https://github.com/$GH_USER/my-new-project.git
cd my-new-project

# -- 或 -- 将现有本地目录推送到新仓库
cd /path/to/existing/project
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/$GH_USER/my-new-project.git
git push -u origin main
```

在组织下创建：

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/orgs/my-org/repos \
  -d '{"name": "my-new-project", "private": false}'
```

### 从模板创建

**使用 gh：**

```bash
gh repo create my-new-app --template owner/template-repo --public --clone
```

**使用 curl：**

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/owner/template-repo/generate \
  -d '{"owner": "'"$GH_USER"'", "name": "my-new-app", "private": false}'
```

### Fork 仓库

**使用 gh：**

```bash
gh repo fork owner/repo-name --clone
```

**使用 git + curl：**

```bash
# 通过 API 创建 fork
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/owner/repo-name/forks

# 等待片刻让 GitHub 创建它，然后克隆
sleep 3
git clone https://github.com/$GH_USER/repo-name.git
cd repo-name

# 将原始仓库添加为 "upstream" 远程
git remote add upstream https://github.com/owner/repo-name.git
```

### 保持 Fork 同步

```bash
# 纯 git — 到处工作
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

**使用 gh（快捷方式）：**

```bash
gh repo sync $GH_USER/repo-name
```

### 仓库信息

**使用 gh：**

```bash
gh repo view owner/repo-name
gh repo list --limit 20
gh search repos "machine learning" --language python --sort stars
```

**使用 curl：**

```bash
# 查看仓库详情
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO \
  | python3 -c "
import sys, json
r = json.load(sys.stdin)
print(f\"Name: {r['full_name']}\")
print(f\"Description: {r['description']}\")
print(f\"Stars: {r['stargazers_count']}  Forks: {r['forks_count']}\")
print(f\"Default branch: {r['default_branch']}\")
print(f\"Language: {r['language']}\")"

# 列出你的仓库
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/user/repos?per_page=20&sort=updated" \
  | python3 -c "
import sys, json
for r in json.load(sys.stdin):
    vis = 'private' if r['private'] else 'public'
    print(f\"  {r['full_name']:40}  {vis:8}  {r.get('language', ''):10}  ★{r['stargazers_count']}\")"

# 搜索仓库
curl -s \
  "https://api.github.com/search/repositories?q=machine+learning+language:python&sort=stars&per_page=10" \
  | python3 -c "
import sys, json
for r in json.load(sys.stdin)['items']:
    print(f\"  {r['full_name']:40}  ★{r['stargazers_count']:6}  {r['description'][:60] if r['description'] else ''}\")"
```

### 仓库设置

**使用 gh：**

```bash
gh repo edit --description "Updated description" --visibility public
gh repo edit --enable-wiki=false --enable-issues=true
gh repo edit --default-branch main
gh repo edit --add-topic "machine-learning,python"
gh repo edit --enable-auto-merge
```

**使用 curl：**

```bash
curl -s -X PATCH \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO \
  -d '{
    "description": "Updated description",
    "has_wiki": false,
    "has_issues": true,
    "allow_auto_merge": true
  }'

# 更新主题
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.mercy-preview+json" \
  https://api.github.com/repos/$OWNER/$REPO/topics \
  -d '{"names": ["machine-learning", "python", "automation"]}'
```

### 分支保护

```bash
# 查看当前保护
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/branches/main/protection

# 设置分支保护
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/branches/main/protection \
  -d '{
    "required_status_checks": {
      "strict": true,
      "contexts": ["ci/test", "ci/lint"]
    },
    "enforce_admins": false,
    "required_pull_request_reviews": {
      "required_approving_review_count": 1
    },
    "restrictions": null
  }'
```

### Secrets 管理（GitHub Actions）

**使用 gh：**

```bash
gh secret set API_KEY --body "your-secret-value"
gh secret set SSH_KEY < ~/.ssh/id_rsa
gh secret list
gh secret delete API_KEY
```

**使用 curl：**

Secrets 需要用仓库公钥加密 — 通过 API 更复杂：

```bash
# 获取仓库公钥用于加密 secrets
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/secrets/public-key

# 加密并设置（需要 Python 与 PyNaCl）
python3 -c "
from base64 import b64encode
from nacl import encoding, public
import json, sys

# 获取公钥
key_id = '<key_id_from_above>'
public_key = '<base64_key_from_above>'

# 加密
sealed = public.SealedBox(
    public.PublicKey(public_key.encode('utf-8'), encoding.Base64Encoder)
).encrypt('your-secret-value'.encode('utf-8'))
print(json.dumps({
    'encrypted_value': b64encode(sealed).decode('utf-8'),
    'key_id': key_id
}))"

# 然后 PUT 加密的 secret
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/secrets/API_KEY \
  -d '<上面 python 脚本的输出>'

# 列出 secrets（仅名称，值隐藏）
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/secrets \
  | python3 -c "
import sys, json
for s in json.load(sys.stdin)['secrets']:
    print(f\"  {s['name']:30}  updated: {s['updated_at']}\")"
```

注意：对于 secrets，`gh secret set` 显著更简单。如果需要设置 secrets 且 `gh` 不可用，建议仅为该操作安装它。

### Releases

**使用 gh：**

```bash
gh release create v1.0.0 --title "v1.0.0" --generate-notes
gh release create v2.0.0-rc1 --draft --prerelease --generate-notes
gh release create v1.0.0 ./dist/binary --title "v1.0.0" --notes "Release notes"
gh release list
gh release download v1.0.0 --dir ./downloads
```

**使用 curl：**

```bash
# 创建 release
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/releases \
  -d '{
    "tag_name": "v1.0.0",
    "name": "v1.0.0",
    "body": "## Changelog\n- Feature A\n- Bug fix B",
    "draft": false,
    "prerelease": false,
    "generate_release_notes": true
  }'

# 列出 releases
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/releases \
  | python3 -c "
import sys, json
for r in json.load(sys.stdin):
    tag = r.get('tag_name', 'no tag')
    print(f\"  {tag:15}  {r['name']:30}  {'draft' if r['draft'] else 'published'}\")"

# 上传 release 资产（二进制文件）
RELEASE_ID=<id_from_create_response>
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Content-Type: application/octet-stream" \
  "https://uploads.github.com/repos/$OWNER/$REPO/releases/$RELEASE_ID/assets?name=binary-amd64" \
  --data-binary @./dist/binary-amd64
```

### GitHub Actions 工作流

**使用 gh：**

```bash
gh workflow list
gh run list --limit 10
gh run view <RUN_ID>
gh run view <RUN_ID> --log-failed
gh run rerun <RUN_ID>
gh run rerun <RUN_ID> --failed
gh workflow run ci.yml --ref main
gh workflow run deploy.yml -f environment=staging
```

**使用 curl：**

```bash
# 列出工作流
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/workflows \
  | python3 -c "
import sys, json
for w in json.load(sys.stdin)['workflows']:
    print(f\"  {w['id']:10}  {w['name']:30}  {w['state']}\")"

# 列出最近运行
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/actions/runs?per_page=10" \
  | python3 -c "
import sys, json
for r in json.load(sys.stdin)['workflow_runs']:
    print(f\"  Run {r['id']}  {r['name']:30}  {r['conclusion'] or r['status']}\")"

# 下载失败运行日志
RUN_ID=<run_id>
curl -s -L \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/runs/$RUN_ID/logs \
  -o /tmp/ci-logs.zip
cd /tmp && unzip -o ci-logs.zip -d ci-logs

# 重新运行失败的工作流
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/runs/$RUN_ID/rerun

# 仅重新运行失败作业
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/runs/$RUN_ID/rerun-failed-jobs

# 手动触发工作流（workflow_dispatch）
WORKFLOW_ID=<workflow_id_or_filename>
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/workflows/$WORKFLOW_ID/dispatches \
  -d '{"ref": "main", "inputs": {"environment": "staging"}}'
```

### Gists

**使用 gh：**

```bash
gh gist create script.py --public --desc "Useful script"
gh gist list
```

**使用 curl：**

```bash
# 创建 gist
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/gists \
  -d '{
    "description": "Useful script",
    "public": true,
    "files": {
      "script.py": {"content": "print(\"hello\")"}
    }
  }'

# 列出你的 gists
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/gists \
  | python3 -c "
import sys, json
for g in json.load(sys.stdin):
    files = ', '.join(g['files'].keys())
    print(f\"  {g['id']}  {g['description'] or '(no desc)':40}  {files}\")"
```

## 三、Issue 管理

### 查看 Issues

**使用 gh：**

```bash
gh issue list
gh issue list --state open --label "bug"
gh issue list --assignee @me
gh issue list --search "authentication error" --state all
gh issue view 42
```

**使用 curl：**

```bash
# 列出开放 issues
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/issues?state=open&per_page=20" \
  | python3 -c "
import sys, json
for i in json.load(sys.stdin):
    if 'pull_request' not in i:  # GitHub API 也在 /issues 中返回 PRs
        labels = ', '.join(l['name'] for l in i['labels'])
        print(f\"#{i['number']:5}  {i['state']:6}  {labels:30}  {i['title']}\")"

# 按标签过滤
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/issues?state=open&labels=bug&per_page=20" \
  | python3 -c "
import sys, json
for i in json.load(sys.stdin):
    if 'pull_request' not in i:
        print(f\"#{i['number']}  {i['title']}\")"

# 查看特定 issue
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42 \
  | python3 -c "
import sys, json
i = json.load(sys.stdin)
labels = ', '.join(l['name'] for l in i['labels'])
assignees = ', '.join(a['login'] for a in i['assignees'])
print(f\"#{i['number']}: {i['title']}\")
print(f\"State: {i['state']}  Labels: {labels}  Assignees: {assignees}\")
print(f\"Author: {i['user']['login']}  Created: {i['created_at']}\")
print(f\"\n{i['body']}\")"

# 搜索 issues
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/issues?q=authentication+error+repo:$OWNER/$REPO" \
  | python3 -c "
import sys, json
for i in json.load(sys.stdin)['items']:
    print(f\"#{i['number']}  {i['state']:6}  {i['title']}\")"
```

### 创建 Issues

**使用 gh：**

```bash
gh issue create \
  --title "Login redirect ignores ?next= parameter" \
  --body "## Description
After logging in, users always land on /dashboard.

## Steps to Reproduce
1. Navigate to /settings while logged out
2. Get redirected to /login?next=/settings
3. Log in
4. Actual: redirected to /dashboard (should go to /settings)

## Expected Behavior
Respect the ?next= query parameter." \
  --label "bug,backend" \
  --assignee "username"
```

**使用 curl：**

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues \
  -d '{
    "title": "Login redirect ignores ?next= parameter",
    "body": "## Description\nAfter logging in, users always land on /dashboard.\n\n## Steps to Reproduce\n1. Navigate to /settings while logged out\n2. Get redirected to /login?next=/settings\n3. Log in\n4. Actual: redirected to /dashboard\n\n## Expected Behavior\nRespect the ?next= query parameter.",
    "labels": ["bug", "backend"],
    "assignees": ["username"]
  }'
```

### Bug 报告模板

```
## Bug Description
<What's happening>

## Steps to Reproduce
1. <step>
2. <step>

## Expected Behavior
<What should happen>

## Actual Behavior
<What actually happens>

## Environment
- OS: <os>
- Version: <version>
```

### 功能请求模板

```
## Feature Description
<What you want>

## Motivation
<Why this would be useful>

## Proposed Solution
<How it could work>

## Alternatives Considered
<Other approaches>
```

### 管理 Issues

#### 添加/移除标签

**使用 gh：**

```bash
gh issue edit 42 --add-label "priority:high,bug"
gh issue edit 42 --remove-label "needs-triage"
```

**使用 curl：**

```bash
# 添加标签
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42/labels \
  -d '{"labels": ["priority:high", "bug"]}'

# 移除标签
curl -s -X DELETE \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42/labels/needs-triage

# 列出仓库中可用标签
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/labels \
  | python3 -c "
import sys, json
for l in json.load(sys.stdin):
    print(f\"  {l['name']:30}  {l.get('description', '')}\")"
```

#### 分配

**使用 gh：**

```bash
gh issue edit 42 --add-assignee username
gh issue edit 42 --add-assignee @me
```

**使用 curl：**

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42/assignees \
  -d '{"assignees": ["username"]}'
```

#### 评论

**使用 gh：**

```bash
gh issue comment 42 --body "Investigated — root cause is in auth middleware. Working on a fix."
```

**使用 curl：**

```bash
curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42/comments \
  -d '{"body": "Investigated — root cause is in auth middleware. Working on a fix."}'
```

#### 关闭和重新打开

**使用 gh：**

```bash
gh issue close 42
gh issue close 42 --reason "not planned"
gh issue reopen 42
```

**使用 curl：**

```bash
# 关闭
curl -s -X PATCH \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42 \
  -d '{"state": "closed", "state_reason": "completed"}'

# 重新打开
curl -s -X PATCH \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/issues/42 \
  -d '{"state": "open"}'
```

#### 链接 Issues 到 PRs

当 PR 合并时，issues 自动关闭，如果正文中有正确关键词：

```
Closes #42
Fixes #42
Resolves #42
```

从 issue 创建分支：

**使用 gh：**

```bash
gh issue develop 42 --checkout
```

**使用 git（手动等效）：**

```bash
git checkout main && git pull origin main
git checkout -b fix/issue-42-login-redirect
```

### Issue 分类工作流

当被要求分类 issues 时：

1. **列出未分类 issues：**

```bash
# 使用 gh
gh issue list --label "needs-triage" --state open

# 使用 curl
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/issues?labels=needs-triage&state=open" \
  | python3 -c "
import sys, json
for i in json.load(sys.stdin):
    if 'pull_request' not in i:
        print(f\"#{i['number']}  {i['title']}\")"
```

2. **阅读和分类**每个 issue（查看详情，理解 bug/功能）

3. **应用标签和优先级**（见上面管理 Issues）

4. **分配**如果所有者清楚

5. **必要时用分类注释评论**

### 批量操作

对于批量操作，结合 API 调用与 shell 脚本：

**使用 gh：**

```bash
# 关闭所有特定标签的 issues
gh issue list --label "wontfix" --json number --jq '.[].number' | \
  xargs -I {} gh issue close {} --reason "not planned"
```

**使用 curl：**

```bash
# 列出有标签的 issue 编号，然后关闭每个
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/issues?labels=wontfix&state=open" \
  | python3 -c "import sys,json; [print(i['number']) for i in json.load(sys.stdin)]" \
  | while read num; do
    curl -s -X PATCH \
      -H "Authorization: token $GITHUB_TOKEN" \
      https://api.github.com/repos/$OWNER/$REPO/issues/$num \
      -d '{"state": "closed", "state_reason": "not_planned"}'
    echo "Closed #$num"
  done
```

## 四、PR 工作流

### 分支创建

这部分是纯 `git` — 两种方式相同：

```bash
# 确保你最新
git fetch origin
git checkout main && git pull origin main

# 创建并切换到新分支
git checkout -b feat/add-user-authentication
```

分支命名约定：
- `feat/description` — 新功能
- `fix/description` — bug 修复
- `refactor/description` — 代码重构
- `docs/description` — 文档
- `ci/description` — CI/CD 变更

### 提交

使用代理的文件工具（`write_file`、`patch`）进行更改，然后提交：

```bash
# 暂存特定文件
git add src/auth.py src/models/user.py tests/test_auth.py

# 用约定式提交消息提交
git commit -m "feat: add JWT-based user authentication

- Add login/register endpoints
- Add User model with password hashing
- Add auth middleware for protected routes
- Add unit tests for auth flow"
```

提交消息格式（约定式提交）：
```
type(scope): short description

Longer explanation if needed. Wrap at 72 characters.
```

类型：`feat`、`fix`、`refactor`、`docs`、`test`、`ci`、`chore`、`perf`

### 推送和创建 PR

#### 推送分支（两种方式相同）

```bash
git push -u origin HEAD
```

#### 创建 PR

**使用 gh：**

```bash
gh pr create \
  --title "feat: add JWT-based user authentication" \
  --body "## Summary
- Adds login and register API endpoints
- JWT token generation and validation

## Test Plan
- [ ] Unit tests pass

Closes #42"
```

选项：`--draft`、`--reviewer user1,user2`、`--label "enhancement"`、`--base develop`

**使用 git + curl：**

```bash
BRANCH=$(git branch --show-current)

curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/$OWNER/$REPO/pulls \
  -d "{
    \"title\": \"feat: add JWT-based user authentication\",
    \"body\": \"## Summary\nAdds login and register API endpoints.\n\nCloses #42\",
    \"head\": \"$BRANCH\",
    \"base\": \"main\"
  }"
```

响应 JSON 包括 PR `number` — 保存它用于后续命令。

创建为草稿，在 JSON 正文中添加 `"draft": true`。

### 监控 CI 状态

#### 检查 CI 状态

**使用 gh：**

```bash
# 一次性检查
gh pr checks

# 监视直到所有检查完成（每 10 秒轮询）
gh pr checks --watch
```

**使用 git + curl：**

```bash
# 获取当前分支的最新提交 SHA
SHA=$(git rev-parse HEAD)

# 查询组合状态
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/commits/$SHA/status \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
print(f\"Overall: {data['state']}\")
for s in data.get('statuses', []):
    print(f\"  {s['context']}: {s['state']} - {s.get('description', '')}\")"

# 也检查 GitHub Actions 检查运行（单独端点）
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/commits/$SHA/check-runs \
  | python3 -c "
import sys, json
data = json.load(sys.stdin)
for cr in data.get('check_runs', []):
    print(f\"  {cr['name']}: {cr['status']} / {cr['conclusion'] or 'pending'}\")"
```

#### 轮询直到完成（git + curl）

```bash
# 简单轮询循环 — 每 30 秒检查一次，最多 10 分钟
SHA=$(git rev-parse HEAD)
for i in $(seq 1 20); do
  STATUS=$(curl -s \
    -H "Authorization: token $GITHUB_TOKEN" \
    https://api.github.com/repos/$OWNER/$REPO/commits/$SHA/status \
    | python3 -c "import sys,json; print(json.load(sys.stdin)['state'])")
  echo "Check $i: $STATUS"
  if [ "$STATUS" = "success" ] || [ "$STATUS" = "failure" ] || [ "$STATUS" = "error" ]; then
    break
  fi
  sleep 30
done
```

### 自动修复 CI 失败

当 CI 失败时，诊断并修复。这个循环与两种认证方法都工作。

#### 步骤 1：获取失败详情

**使用 gh：**

```bash
# 列出此分支的最近工作流运行
gh run list --branch $(git branch --show-current) --limit 5

# 查看失败日志
gh run view <RUN_ID> --log-failed
```

**使用 git + curl：**

```bash
BRANCH=$(git branch --show-current)

# 列出此分支的工作流运行
curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/$OWNER/$REPO/actions/runs?branch=$BRANCH&per_page=5" \
  | python3 -c "
import sys, json
runs = json.load(sys.stdin)['workflow_runs']
for r in runs:
    print(f\"Run {r['id']}: {r['name']} - {r['conclusion'] or r['status']}\")"

# 获取失败作业日志（下载为 zip，解压，读取）
RUN_ID=<run_id>
curl -s -L \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/actions/runs/$RUN_ID/logs \
  -o /tmp/ci-logs.zip
cd /tmp && unzip -o ci-logs.zip -d ci-logs && cat ci-logs/*.txt
```

#### 步骤 2：修复和推送

识别问题后，使用文件工具（`patch`、`write_file`）修复：

```bash
git add <fixed_files>
git commit -m "fix: resolve CI failure in <check_name>"
git push
```

#### 步骤 3：验证

使用上面第 4 节的命令重新检查 CI 状态。

#### 自动修复循环模式

当被要求自动修复 CI 时，遵循这个循环：

1. 检查 CI 状态 → 识别失败
2. 读取失败日志 → 理解错误
3. 使用 `read_file` + `patch`/`write_file` → 修复代码
4. `git add . && git commit -m "fix: ..." && git push`
5. 等待 CI → 重新检查状态
6. 如果仍失败则重复（最多 3 次尝试，然后询问用户）

### 合并

**使用 gh：**

```bash
# Squash 合并 + 删除分支（对功能分支最干净）
gh pr merge --squash --delete-branch

# 启用自动合并（所有检查通过时合并）
gh pr merge --auto --squash --delete-branch
```

**使用 git + curl：**

```bash
PR_NUMBER=<number>

# 通过 API 合并 PR（squash）
curl -s -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/pulls/$PR_NUMBER/merge \
  -d "{
    \"merge_method\": \"squash\",
    \"commit_title\": \"feat: add user authentication (#$PR_NUMBER)\"
  }"

# 合并后删除远程分支
BRANCH=$(git branch --show-current)
git push origin --delete $BRANCH

# 本地切换回 main
git checkout main && git pull origin main
git branch -d $BRANCH
```

合并方法：`"merge"`（合并提交）、`"squash"`、`"rebase"`

### 启用自动合并（curl）

```bash
# 自动合并需要仓库在设置中启用它。
# 这使用 GraphQL API，因为 REST 不支持自动合并。
PR_NODE_ID=$(curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/$OWNER/$REPO/pulls/$PR_NUMBER \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['node_id'])")

curl -s -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/graphql \
  -d "{\"query\": \"mutation { enablePullRequestAutoMerge(input: {pullRequestId: \\\"$PR_NODE_ID\\\", mergeMethod: SQUASH}) { clientMutationId } }\"}"
```

### 完整工作流示例

```bash
# 1. 从干净的 main 开始
git checkout main && git pull origin main

# 2. 分支
git checkout -b fix/login-redirect-bug

# 3. （代理用文件工具进行代码更改）

# 4. 提交
git add src/auth/login.py tests/test_login.py
git commit -m "fix: correct redirect URL after login

Preserves the ?next= parameter instead of always redirecting to /dashboard."

# 5. 推送
git push -u origin HEAD

# 6. 创建 PR（根据可用内容选择 gh 或 curl）
# ... （见上面第 3 节）

# 7. 监控 CI（见上面第 4 节）

# 8. 绿色时合并（见上面第 6 节）
```

### 有用 PR 命令参考

| 操作 | gh | git + curl |
|------|-----|-----------|
| 列出我的 PR | `gh pr list --author @me` | `curl -s -H "Authorization: token $GITHUB_TOKEN" "https://api.github.com/repos/$OWNER/$REPO/pulls?state=open"` |
| 查看 PR diff | `gh pr diff` | `git diff main...HEAD`（本地）或 `curl -H "Accept: application/vnd.github.diff" ...` |
| 添加评论 | `gh pr comment N --body "..."` | `curl -X POST .../issues/N/comments -d '{"body":"..."}'` |
| 请求审查 | `gh pr edit N --add-reviewer user` | `curl -X POST .../pulls/N/requested_reviewers -d '{"reviewers":["user"]}'` |
| 关闭 PR | `gh pr close N` | `curl -X PATCH .../pulls/N -d '{"state":"closed"}'` |
| 检出某人的 PR | `gh pr checkout N` | `git fetch origin pull/N/head:pr-N && git checkout pr-N` |

## 五、代码分析

### 使用 pygount 检查代码库

分析仓库的代码行数、语言分解、文件数和代码与注释比率。

#### 何时使用

- 用户要求 LOC（代码行数）计数
- 用户想要仓库的语言分解
- 用户询问代码库大小或组成
- 用户想要代码与注释比率
- 一般"这个仓库有多大"问题

#### 先决条件

```bash
pip install --break-system-packages pygount 2>/dev/null || pip install pygount
```

#### 基本摘要（最常见）

获取完整语言分解与文件数、代码行和注释行：

```bash
cd /path/to/repo
pygount --format=summary \
  --folders-to-skip=".git,node_modules,venv,.venv,__pycache__,.cache,dist,build,.next,.tox,.eggs,*.egg-info" \
  .
```

**重要：** 总是使用 `--folders-to-skip` 排除依赖/构建目录，否则 pygount 会爬取它们并花费非常长时间或在大依赖树上挂起。

#### 常见文件夹排除

根据项目类型调整：

```bash
# Python 项目
--folders-to-skip=".git,venv,.venv,__pycache__,.cache,dist,build,.tox,.eggs,.mypy_cache"

# JavaScript/TypeScript 项目
--folders-to-skip=".git,node_modules,dist,build,.next,.cache,.turbo,coverage"

# 通用捕获所有
--folders-to-skip=".git,node_modules,venv,.venv,__pycache__,.cache,dist,build,.next,.tox,vendor,third_party"
```

#### 按特定语言过滤

```bash
# 只计数 Python 文件
pygount --suffix=py --format=summary .

# 只计数 Python 和 YAML
pygount --suffix=py,yaml,yml --format=summary .
```

#### 详细逐文件输出

```bash
# 默认格式显示逐文件分解
pygount --folders-to-skip=".git,node_modules,venv" .

# 按代码行排序（通过 sort 管道）
pygount --folders-to-skip=".git,node_modules,venv" . | sort -t$'\t' -k1 -nr | head -20
```

#### 输出格式

```bash
# 摘要表（默认推荐）
pygount --format=summary .

# JSON 输出用于程序化使用
pygount --format=json .

# 管道友好：Language, file count, code, docs, empty, string
pygount --format=summary . 2>/dev/null
```

#### 解释结果

摘要表列：
- **Language** — 检测到的编程语言
- **Files** — 该语言的文件数
- **Code** — 实际代码行（可执行/声明性）
- **Comment** — 注释或文档行
- **%** — 总数的百分比

特殊伪语言：
- `__empty__` — 空文件
- `__binary__` — 二进制文件（图像、编译等）
- `__generated__` — 自动生成的文件（启发式检测）
- `__duplicate__` — 内容相同的文件
- `__unknown__` — 无法识别的文件类型

#### 陷阱

1. **总是排除 .git、node_modules、venv** — 没有 `--folders-to-skip`，pygount 会爬取所有内容，可能在大依赖树上花费几分钟或挂起。
2. **Markdown 显示 0 代码行** — pygount 将所有 Markdown 内容分类为注释，不是代码。这是预期行为。
3. **JSON 文件显示低代码计数** — pygount 可能保守地计数 JSON 行。对于准确的 JSON 行计数，直接使用 `wc -l`。
4. **大单体仓库** — 对非常大的仓库，考虑使用 `--suffix` 针对特定语言，而不是扫描所有内容。

## 六、常见陷阱汇总

### 认证
- **git push 要求密码** — GitHub 禁用了密码认证。使用个人访问令牌或 SSH
- **权限被拒绝** — 令牌可能缺少 `repo` 范围
- **认证失败** — 缓存凭据可能过期
- **SSH 连接拒绝** — 尝试通过端口 443 的 SSH

### 仓库管理
- **克隆大仓库慢** — 使用 `--depth 1` 浅克隆
- **Fork 不同步** — 定期运行 `git fetch upstream && git merge upstream/main`
- **Secrets 需要加密** — 使用 `gh secret set` 显著更简单

### Issue 管理
- **GitHub API 也在 /issues 中返回 PRs** — 过滤掉 `'pull_request' not in i`
- **批量操作** — 结合 API 调用与 shell 脚本

### PR 工作流
- **CI 失败** — 使用自动修复循环模式（最多 3 次尝试）
- **合并方法** — `squash` 对功能分支最干净
- **自动合并** — 需要仓库在设置中启用

### 代码分析
- **总是排除依赖目录** — 否则 pygount 会挂起
- **Markdown 显示 0 代码行** — 预期行为
- **大单体仓库** — 使用 `--suffix` 针对特定语言

## 七、参考资源

- GitHub 认证：`github-auth` skill
- GitHub Issues：`github-issues` skill
- GitHub PR 工作流：`github-pr-workflow` skill
- GitHub 仓库管理：`github-repo-management` skill
- 代码库检查：`codebase-inspection` skill
- GitHub API 文档：https://docs.github.com/en/rest
