---
name: browser-automation
description: "浏览器自动化统一入口：数据采集、桌面控制、AppleScript、MCP、安全绕过。"
tags: [browser, automation, scraping, chrome, applescript, mcp]
---

# 浏览器自动化

统一入口：覆盖网页数据采集、桌面级控制、AppleScript 直连、MCP 自然语言、安全沙箱绕过。

## 场景决策树

```
开始
├─ 公开页面正文？ → web_extract（最快）
├─ 需要找 URL？ → web_search（搜索引擎）
├─ 需要 JS 渲染/动态页面？ → browser_* 工具链（Hermes 内置）
├─ 需要登录态/Cookie 持久化？ → agent-browser
├─ 需要抓 API 接口定义？ → agent-browser（HAR 录制）
├─ 反爬虫严格？ → agent-browser + 云浏览器（Browserbase/Kernel）
├─ 桌面级复杂交互（原生对话框）？ → computer_use
├─ 只需读取 Chrome 标签页信息？ → AppleScript（最轻量）
├─ 需要自然语言控制浏览器？ → browser-mcp-agent（Playwright）
└─ 安全沙箱/渲染隔离绕过？ → Sensor-Receiver 模式（Chrome Extension）
```

## 一、网页数据采集

### 1.1 web_extract - 公开页面正文

**适用**：静态页面、文章、文档、PDF  
**不适用**：需要登录、JS 渲染、反爬虫严格

```python
web_extract(urls=["https://example.com/article"])
web_extract(urls=[...], char_limit=30000)
```

### 1.2 web_search - 搜索引擎发现

```python
web_search(query="site:github.com agent-browser", limit=10)
```

### 1.3 browser_* 工具链 - JS 渲染页面

**适用**：SPA、动态加载、简单交互  
**不适用**：需要长期保持登录态、需要抓 API

```python
browser_navigate(url="https://example.com")
browser_snapshot(full=True)
browser_click(ref="@e5")
browser_type(ref="@e3", text="search query")
browser_scroll(direction="down")
```

### 1.4 agent-browser - 登录态 + API 抓取

**适用**：需要登录的 SaaS、抓取后端 API 接口、反爬虫  
**不适用**：简单公开页面（杀鸡用牛刀）

**安装状态**：✅ v0.27.0 已安装，本地免费

#### 核心命令

```bash
# 基础控制
agent-browser open <url>
agent-browser snapshot              # Accessibility Tree
agent-browser click @e2
agent-browser fill @e3 "text"
agent-browser screenshot [path]

# Cookie 管理
agent-browser cookies get
agent-browser cookies set <name> <val>
agent-browser cookies set --curl <file>  # 从 cURL 导入

# 会话持久化
agent-browser --session-name myapp open example.com  # 自动保存/恢复
agent-browser --profile Default open gmail.com       # 复用 Chrome 登录态

# HAR 网络抓包
agent-browser network har start
agent-browser network har stop output.har

# 网络请求查看
agent-browser network requests --filter api
agent-browser network requests --type xhr,fetch
```

#### 典型工作流

**A. 登录态爬取**
```bash
agent-browser --session-name myapp open https://saas.example.com
# 手动登录后...
agent-browser cookies get > cookies.json

# 后续自动恢复
agent-browser --session-name myapp open https://saas.example.com/dashboard
```

**B. 抓取 API 接口**
```bash
agent-browser network har start
agent-browser open https://example.com
agent-browser click @e5
agent-browser wait 2000
agent-browser network har stop api.har
```

**C. 多会话并行**
```bash
agent-browser --session data open https://data.example.com
agent-browser --session admin open https://admin.example.com
agent-browser session list
```

### 1.5 computer_use - 桌面级交互

**适用**：原生对话框、复杂 JS 应用、需要视觉判断  
**不适用**：简单网页（太重）

```python
computer_use(action="capture", mode="som")  # 截图+元素标注
computer_use(action="click", element=14)    # 点击元素
computer_use(action="type", text="query")   # 输入文本
```

## 二、AppleScript 直连 Chrome（macOS 专属）

**适用**：只需读取标签页信息、导航、执行简单 JS  
**不适用**：复杂 DOM 操作、网络请求监控

**核心优势**：无需端口绑定，避免 TCC 权限问题，系统级 API 稳定性高。

### 标准命令集

```bash
# 读取当前标签页
osascript -e 'tell application "Google Chrome" to get {url, title} of active tab of window 1'

# 读取所有标签页
osascript -e 'tell application "Google Chrome" to get {url, title} of every tab of window 1'

# 导航到新页面
osascript -e 'tell application "Google Chrome" to set URL of active tab of window 1 to "https://example.com"'

# 执行 JavaScript
osascript -e 'tell application "Google Chrome" to execute active tab of window 1 javascript "document.title"'

# 获取页面内容
osascript -e 'tell application "Google Chrome" to get the content of active tab of window 1'

# 关闭/刷新标签页
osascript -e 'tell application "Google Chrome" to close active tab of window 1'
osascript -e 'tell application "Google Chrome" to reload active tab of window 1'
```

### 遍历所有窗口和标签页

```bash
osascript << 'EOF'
tell application "Google Chrome"
    set output to ""
    repeat with w in windows
        repeat with t in tabs of w
            set output to output & (URL of t) & " | " & (title of t) & linefeed
        end repeat
    end repeat
    return output
end tell
EOF
```

### AppleScript vs MCP 对比

| 特性 | AppleScript | MCP (chrome-devtools) |
|------|-------------|----------------------|
| 端口需求 | ❌ 不需要 | ✅ 必须 9222 |
| TCC 权限 | ✅ 有豁免 | ⚠️ 受限制 |
| 功能范围 | 基础信息 + JS | 完整 DevTools |
| 稳定性 | ✅ 高 | ⚠️ 中等 |
| 配置复杂度 | 低 | 高 |

### 故障排查

**Chrome 显示"新配置"/数据丢失**：
```bash
# 诊断
ls -la ~/Library/Application\ Support/Google/Chrome/Profile\ Preferences

# 恢复（从 ChromeDebug 备份）
cp ~/Library/Application\ Support/Google/ChromeDebug/Default/Preferences \
   ~/Library/Application\ Support/Google/Chrome/Profile\ Preferences
```

## 三、MCP 自然语言浏览器控制

**适用**：用自然语言控制浏览器（导航、点击、填表、截图、提取）  
**不适用**：需要高性能批量操作

### 安装

```bash
git clone https://github.com/Shubhamsaboo/awesome-llm-apps.git
cd awesome-llm-apps/mcp_ai_agents/browser_mcp_agent
pip install -r requirements.txt
export OPENAI_API_KEY=your-key
```

### 使用示例

```
"Go to github.com/Shubhamsaboo/awesome-llm-apps"
"Click on mcp_ai_agents"
"Scroll down and summarize the github readme"
"Fill the search box with 'hello world' and press enter"
```

### 本地 Ollama 模型

```bash
export MODEL=ollama/llama3.3
streamlit run main.py
```

## 四、安全沙箱绕过（Sensor-Receiver 模式）

**适用**：标准自动化（CDP/Puppeteer）或视觉感知（截图）因安全策略或硬件加速渲染隔离而失败时。

### 核心方法论

当遇到以下情况时使用：
- **Security Sandbox Block**：无法注入代码（fetch/XHR 被阻止）
- **Rendering Isolation**：能看到 DOM 但看不到像素（GPU 加速"黑洞"）

### 三层架构

```
1. Sensor（Chrome Extension）
   - 在浏览器执行上下文内作为内部代理
   - 访问 chrome.* APIs（tabs, scripting, webRequest）
   - 通过 Service Worker 桥接数据

2. Bridge（WebSocket/Native Messaging）
   - 低延迟双向 JSON 数据流
   - 从浏览器内部状态到本地环境

3. Receiver（本地进程）
   - Python/Node.js 消费数据流
   - 实时分析、日志、编排
```

### 实施工作流

1. **识别失败模式**：Security Sandbox 还是 Rendering Isolation？
2. **部署 Sensor**：在"开发者模式"加载自定义 Chrome Extension
3. **初始化 Receiver**：启动本地 WebSocket 服务器
4. **监控流**：用 receiver 观察实时遥测

### 常见陷阱与解决方案

| 问题 | 解决方案 |
|------|---------|
| fetch/XHR 被阻止 | 用 extension 的 `background.js` 代理所有网络请求 |
| 截图黑屏（GPU 隔离） | 用 `content_script` 通过 Accessibility Tree 提取 DOM/Video 状态 |
| MCP DevTools 连接失败 | 通常不是网络问题，是 Chrome 没带 `--remote-debugging-port` 启动。修复：`killall -9 "Google Chrome"` 然后用 AppleScript 重启 |
| MCP 端口绑定失败 | 更新配置：`hermes config set mcpServers.<name>.args` 使用 `--browser-url http://localhost:9222` |
| 诊断端口绑定 | 用 `lsof -i :<port>` 检查端口是否真的在监听，`curl http://localhost:9222/json/version` 验证 |

## 五、常见陷阱汇总

### web_extract
- 页面过大时会被截断，需要设置 `char_limit`
- 不支持需要登录的页面

### browser_*
- 每次都是新会话，Cookie 不持久
- 无法录制网络请求

### agent-browser
- v0.27.0 的 HAR 命令需要 `network har` 前缀
- 会话冲突时用 `agent-browser close --all`
- Cookie 有有效期，长期不用需重新登录
- 云浏览器（Browserbase/Kernel）需要付费 API Key

### computer_use
- 最重的方案，只在必要时使用
- 需要 cua-driver 安装

### AppleScript
- 首次使用需授权"辅助功能"访问 Chrome
- `window 1` 是最前面的窗口
- Chrome 未运行时 AppleScript 会报错

### MCP Agent
- 需要 API key（OpenAI/Anthropic/Ollama）
- 浏览器状态不会在会话间持久化
- 需要 working display（headless 模式可能需要额外配置）

## 六、组合使用示例

**目标**：爬取需要登录的 SaaS 系统，同时获取 API 接口

```bash
# 1. 登录并保存状态
agent-browser --session-name myapp open https://saas.example.com/login
# 手动登录...

# 2. 开始录制 API
agent-browser network har start

# 3. 执行操作
agent-browser open https://saas.example.com/dashboard
agent-browser click @e10
agent-browser wait 2000

# 4. 停止录制
agent-browser network har stop api.har

# 5. 提取数据
agent-browser snapshot
```

**目标**：监控用户当前浏览的 YouTube 视频

```bash
# 最轻量方式：AppleScript
osascript -e 'tell application "Google Chrome" to get {url, title} of active tab of window 1'
# 输出: https://www.youtube.com/watch?v=xxx | 视频标题
```

## 七、参考资源

- agent-browser 官方：https://agent-browser.dev/
- agent-browser 内置 skill：`agent-browser skills get core --full`
- AppleScript 官方文档：https://developer.apple.com/library/archive/documentation/AppleScript/
- MCP-Agent GitHub：https://github.com/lastmile-ai/mcp-agent
- Playwright 文档：https://playwright.dev/
