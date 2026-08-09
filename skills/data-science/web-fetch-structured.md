---
name: web-fetch-structured
description: 结构化网页内容抓取 - 从 Grok Build 的 web_fetch 工具提取的工程化实践，包含缓存、SSRF防护和域名白名单机制。
version: 1.0.0
tags: [web, fetch, scraping, structured-data]
---

# Web Fetch 结构化抓取技能

## 触发条件
当用户需要：
- 从网页提取结构化内容（文章、数据、API响应）
- 批量获取多个页面的信息
- 需要缓存机制避免重复请求
- 访问可能受限的域名

## 核心实现

### 1. 基础抓取函数（`scripts/web_fetch.sh`）

脚本已大幅升级，详见 `skill_view(name='web-fetch-structured', file_path='scripts/web_fetch.sh')`。关键特性：

| 特性 | 说明 |
|------|------|
| SSRF防护 | 拒绝 localhost/127.x/10.x/172.16-31.x/192.168.x/::1 等内网地址 |
| Python HTML解析器 | `html_to_text()` 用 Python HTMLParser，支持 h1-h3/p/br/li/a 标签语义转换，跳过 script/style |
| macOS兼容缓存 | `stat -f %m` (BSD) / `stat -c %Y` (Linux) 自动适配 |
| 4种输出格式 | text（纯文本）/ json（结构化元数据）/ markdown（基础Markdown）/ raw（原始HTML） |
| 批量抓取 | `batch <urls_file> [format]`，支持注释行跳过，统计成功/失败数 |
| 缓存管理 | `cache-clear` / `cache-info` 命令 |

### CLI用法
```bash
# 单页面抓取
~/.hermes/skills/web-fetch-structured/scripts/web_fetch.sh fetch <url> [text|json|markdown|raw]

# 批量抓取
~/.hermes/skills/web-fetch-structured/scripts/web_fetch.sh batch <urls_file> [format]

# 缓存管理
~/.hermes/skills/web-fetch-structured/scripts/web_fetch.sh cache-clear
~/.hermes/skills/web-fetch-structured/scripts/web_fetch.sh cache-info
```

### 2. 域名白名单配置
创建 `~/.hermes/config/web_fetch_whitelist.txt`：
```
# 允许抓取的域名（每行一个）
github.com
stackoverflow.com
docs.python.org
rust-lang.org
hermes-agent.nousresearch.com
```
不在白名单的域名会打印 WARNING 但仍会继续抓取。

### 3. HTML转Markdown增强
脚本内置 `html_to_markdown()`，在 `html_to_text` 基础上追加：
- h1/h2 → ## / ### → #### 标题层级偏移（适配 Markdown 嵌套）
- li → • 列表符号转换

## 使用示例

### 单页面抓取
```bash
~/.hermes/skills/web-fetch-structured/scripts/web_fetch.sh fetch https://example.com text
```

### 批量抓取URL列表
```bash
# 创建URL列表文件
echo "https://github.com/nousresearch/hermes-agent" > /tmp/urls.txt
echo "https://docs.python.org/3/library/" >> /tmp/urls.txt

# 执行批量抓取
~/.hermes/skills/web-fetch-structured/scripts/web_fetch.sh batch /tmp/urls.txt
```

### 清理缓存
```bash
~/.hermes/skills/web-fetch-structured/scripts/web_fetch.sh cache-clear
```

## 与 Hermes Agent 集成

在 Hermes Agent 中使用时，可以通过 `terminal` 工具调用：

```python
from hermes_tools import terminal

result = terminal(
    command="~/.hermes/skills/web-fetch-structured/scripts/web_fetch.sh fetch https://example.com text",
    timeout=30
)
print(result["output"])
```

## 注意事项

1. **SSRF防护**：始终验证URL scheme，只允许http/https
2. **缓存策略**：默认1小时缓存，避免重复请求
3. **UA标识**：使用 `HermesAgent/1.0` 标识，便于网站识别
4. **超时设置**：curl --max-time 30秒，防止卡死
5. **白名单机制**：可选启用，增强安全性

## 扩展点

- 可集成 BeautifulSoup（Python）进行更复杂的HTML解析
- 可扩展支持 JSON API 的自动格式化
- 可添加代理支持（`curl -x proxy_url`）
- 可集成 rate limiting 避免被封禁