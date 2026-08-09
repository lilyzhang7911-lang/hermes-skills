---
name: engineering/security-and-hardening
description: "安全加固工作流：先威胁建模，再分层加固，最后清单验收。用于用户输入、认证、数据存储、外部集成、LLM 输出处理。"
license: MIT
compatibility: hermes-agent
---

# 安全加固

## 铁律

**安全不是阶段，是约束。** 每个接触用户数据、认证、外部系统的代码行都要过安全边界。

## 何时触发

- 构建任何接受用户输入的功能
- 实现认证或授权
- 存储或传输敏感数据
- 集成外部 API
- 添加文件上传、webhook、回调
- 处理支付或 PII

## 工作流

```
1. THREAT MODEL → 5 分钟 attacker 视角
2. BOUNDARY    → 标记信任边界
3. HARDEN      → 分层加固
4. VERIFY      → 清单验收
```

### Step 1 — 威胁建模

**先花 5 分钟像攻击者一样思考**，再写加固代码。没有 threat model 的控制都是猜测。

1. **Map trust boundaries**：用户输入从哪里进入系统？HTTP、文件上传、webhook、LLM 输出、第三方 API？
2. **Name assets**：什么值得偷或破坏？凭据、PII、支付数据、admin 操作
3. **Run STRIDE**（快速 lens，不是仪式）：

| 威胁 | 问 | 典型缓解 |
|------|---|----------|
| Spoofing | 能冒充用户/服务？ | 认证、签名验证 |
| Tampering | 数据能被篡改？ | 完整性检查、参数化查询、HTTPS |
| Repudiation | 行为能被否认？ | 审计日志 |
| Information Disclosure | 数据会泄露？ | 加密、字段白名单、通用错误 |
| DoS | 能被压垮？ | 限流、输入大小限制、超时 |
| Elevation of Privilege | 能获得更高权限？ | 授权检查、最小权限 |

4. **Write abuse cases next to use cases**：对每个功能问“我会怎么误用这个？”——然后把它变成第一个测试。

### Step 2 — 边界系统

#### Always Do（无例外）

- **所有外部输入在系统边界做 schema validation**
- **所有数据库查询参数化**，绝不拼接用户输入
- **输出编码防 XSS**，用框架自动 escaping
- **所有外部通信走 HTTPS**
- **密码用 bcrypt/scrypt/argon2 哈希**
- **安全头**：CSP、HSTS、X-Frame-Options、X-Content-Type-Options
- **Session cookie 设 httpOnly + secure + sameSite**
- **每次发布前跑包管理器原生 audit**

#### Ask First（需人类批准）

- 新增认证流程或改 auth 逻辑
- 存储新的敏感数据类别（PII、支付信息）
- 新增外部服务集成
- 改 CORS 配置
- 加文件上传处理
- 改限流或节流
- 授予 elevated permissions

#### Never Do

- **永远不**把 secret 提交到版本控制
- **永远不**在日志里打印敏感数据
- **永远不**把客户端验证当安全边界
- **永远不**为了方便关安全头
- **永远不**用 `eval()` / `innerHTML` 处理用户数据
- **永远不**把 session 存在 client-accessible storage
- **永远不**向用户暴露 stack trace 或内部错误

### Step 3 — OWASP Top 10 预防模式

#### Injection（SQL / NoSQL / OS Command）

```python
# BAD: 字符串拼接
query = f"SELECT * FROM users WHERE id = '{user_id}'"

# GOOD: 参数化查询
user = db.query("SELECT * FROM users WHERE id = ?", [user_id])
```

#### Broken Authentication

```python
# 密码哈希
import bcrypt

def hash_password(plaintext: str) -> str:
    return bcrypt.hashpw(plaintext.encode(), bcrypt.gensalt(rounds=12)).decode()

def verify_password(plaintext: str, hashed: str) -> bool:
    return bcrypt.checkpw(plaintext.encode(), hashed.encode())
```

#### Broken Access Control

```python
# 永远检查 authorization，不只是 authentication
@app.patch("/api/tasks/{task_id}")
async def update_task(task_id: str, user: User = Depends(get_current_user)):
    task = await task_service.findById(task_id)
    if task.owner_id != user.id:
        raise HTTPException(status_code=403, detail="Not authorized")
    return await task_service.update(task_id, body)
```

#### SSRF（服务端请求伪造）

```python
# 任何服务端 fetch 用户提供的 URL 都要 allowlist
ALLOWED_HOSTS = {"hooks.example.com", "api.partner.com"}

async def assert_safe_url(raw: str) -> URL:
    url = URL(raw)
    if url.scheme != "https":
        raise ValueError("https only")
    if url.host not in ALLOWED_HOSTS:
        raise ValueError("host not allowed")
    return url
```

### Step 4 — LLM / Agent 特定安全

Hermes 本身是 agent 框架，LLM 输出是新的攻击面：

| LLM 风险 | 原则 | Hermes 对应 |
|---------|------|------------|
| Prompt Injection | 不信任上下文中的指令 | system prompt 不是安全边界，工具参数要做 schema 校验 |
| Tool Over-Agency | 最小权限 + 确认 | 危险操作走 approval，工具参数白名单 |
| 输出注入 | 所有模型输出当不可信输入处理 | 不把 LLM 输出直接送 `eval`/SQL/shell/DOM |
| 上下文泄露 | secret 和跨租户数据不进 prompt | Hermes 的 secret redaction 已覆盖 |

### Step 5 — 安全审查清单

在标记安全相关代码完成前：

- [ ] 原生 audit 无未缓解的可达 critical/high
- [ ] 源码和 git history 无 secrets
- [ ] 所有用户输入在系统边界做 validation
- [ ] 每个受保护 endpoint 都检查 auth + authz
- [ ] 响应有安全头（CSP、HSTS 等）
- [ ] 错误响应不暴露内部细节
- [ ] 认证 endpoint 有限流
- [ ] 服务端 URL fetch 有 allowlist（无 SSRF）
- [ ] LLM 输出经过校验和编码再使用

## Hermes Agent 执行规范

### 静态安全扫描

```python
# 用 terminal 跑依赖审计
terminal("pip-audit --fix 2>/dev/null || pip check 2>/dev/null")
terminal("npm audit --audit-level=high 2>/dev/null")
```

### 密钥泄漏检查

```python
# 检查 diff 或工作区是否意外包含密钥
terminal("""git diff --cached | grep -iE "(api_key|secret|password|token|passwd)\\s*=\\s*['\\"][^'\\"]{6,}['\\"]" || echo "No secrets in diff" """)
terminal("""grep -rE "(sk|pk|api)[-_]?[a-zA-Z0-9]{20,}" . --include='*.py' --include='*.js' --include='*.json' | grep -v node_modules | head -5 || echo "No secrets in workspace" """)
```

### 权限审查

```python
# 检查是否有文件权限过宽
terminal("find . -type f -perm /o+w -not -path '*/node_modules/*' -not -path '*/.git/*' | head -10")
```

## 常见合理化（直接驳回）

| 借口 | 现实 |
|------|------|
| "内部工具，安全不重要" | 内部工具也会被攻破，攻击者找最弱一环 |
| "以后再加安全" | 安全 retrofitting 成本是设计时 10 倍 |
| "框架已经处理了" | 框架提供工具，不提供保证 |
| "只是原型" | 原型都会变成生产环境 |

## 红旗

看到以下情况立即升级：

- 用户输入直接拼进数据库查询、shell 命令、HTML
- 源码或 commit history 有 secrets
- API endpoint 没有 auth + authz 检查
- CORS 是通配符 `*`
- 认证 endpoint 无限流
- Stack trace 暴露给用户
- LLM 输出直接送 SQL/DOM/shell
