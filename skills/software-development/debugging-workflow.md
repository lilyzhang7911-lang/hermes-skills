---
name: debugging-workflow
description: "调试工作流统一入口：方法论、Python 调试、Node.js 调试。"
tags: [debugging, python, nodejs, pdb, debugpy, inspect, cdp]
---

# 调试工作流

统一入口：覆盖调试方法论、Python 调试工具、Node.js 调试工具。

## 场景决策树

```
开始
├─ 需要调试方法论/流程？ → 4 阶段根因调试
├─ 需要调试 Python 代码？ → Python 调试工具
├─ 需要调试 Node.js 代码？ → Node.js 调试工具
└─ 需要调试 Hermes 特定进程？ → Hermes 进程调试
```

## 一、4 阶段根因调试方法论

### 核心原则

**铁律：没有根因调查，不尝试修复。**

如果你没有完成阶段 1，你不能提出修复。

**违反这个过程的精神就是违反调试的精神。**

### 反馈循环规则

反馈循环就是调试工作。在阅读代码构建理论之前，创建或识别一个**紧密的**命令，可以在用户的确切症状上变红，在 bug 修复后变绿。紧密循环是快速、确定性、代理可运行、足够具体以捕获这个 bug——而不仅仅是"不崩溃"。

当干净的复现很难时，花不成比例的努力构建循环。没有红能力循环的猜测是这个 skill 存在以防止的失败模式。

### 何时使用

用于任何技术问题：
- 测试失败
- 生产中的 bug
- 意外行为
- 性能问题
- 构建失败
- 集成问题

**特别在以下情况使用：**
- 时间压力下（紧急情况使猜测诱人）
- "只是一个快速修复"看起来很明显
- 你已经尝试了多个修复
- 之前的修复没有工作
- 你没有完全理解问题

**不要在以下情况跳过：**
- 问题看起来简单（简单 bug 也有根因）
- 你很匆忙（匆忙保证返工）
- 有人想要现在修复（系统化比挣扎更快）

### 阶段 1：根因调查

**在尝试任何修复之前：**

#### 1. 仔细阅读错误消息

- 不要跳过错误或警告
- 它们通常包含确切的解决方案
- 完整阅读堆栈跟踪
- 注意行号、文件路径、错误代码

**行动：** 使用 `read_file` 读取相关源文件。使用 `search_files` 在代码库中查找错误字符串。

#### 2. 构建紧密反馈循环

- 你能用一个命令触发用户的确切症状吗？
- 命令是否因为这个 bug 失败，只有在 bug 修复后才通过？
- 它是否足够快可以重复运行？
- 它是否是确定性的？对于不稳定的 bug，你能将复现率提高到足以调试吗？
- 如果不可复现 → 收集更多数据，不要猜测。

**构建循环的方式——大致按此顺序尝试：**

1. **失败的测试**在到达 bug 的接缝处：单元、集成或端到端。
2. **HTTP 脚本 / curl** 针对运行的开发服务器。
3. **CLI 调用**与 fixture 输入，对预期输出 diff stdout/stderr。
4. **无头浏览器脚本**（Playwright/Puppeteer）断言 DOM、控制台或网络。
5. **重放捕获的跟踪**：HAR、请求负载、事件日志、队列消息或 webhook 主体。
6. **一次性 harness** 启动系统最小有用的切片并调用失败路径。
7. **属性 / 模糊循环** 当 bug 是间歇性错误输出在广泛输入空间上。
8. **二分 harness** 适合 `git bisect run` 当 bug 出现在两个已知状态之间。
9. **差分循环** 比较旧 vs 新版本、两个配置、两个提供者或两个数据集。
10. **人在环中脚本** 仅作为最后手段：脚本化人类步骤并捕获他们的结果，使循环保持结构化。

**一旦存在就收紧循环：**

- 使其更快：缓存设置、缩小范围、跳过不相关的初始化。
- 使信号更锐利：断言确切症状，不是泛型成功。
- 使其更确定性：固定时间、种子随机性、隔离文件系统、冻结网络。

对于非确定性 bug，直接目标是更高的复现率，不是完美。运行触发器 100 次、并行化、添加压力、缩小时间窗口或注入睡眠。50% 不稳定是可调试的；1% 不稳定通常不是。

**行动：** 使用 `terminal` 工具运行紧密循环：

```bash
# 运行特定失败测试
pytest tests/test_module.py::test_name -v

# 或运行脚本化复现
python scripts/repro_bug.py

# 或运行高重复不稳定复现
for i in {1..100}; do pytest tests/test_flake.py::test_name -q || break; done
```

#### 3. 检查最近更改

- 什么更改可能导致这个？
- Git diff、最近提交
- 新依赖、配置更改

**行动：**

```bash
# 最近提交
git log --oneline -10

# 未提交更改
git diff

# 特定文件的更改
git log -p --follow src/problematic_file.py | head -100
```

#### 4. 在多组件系统中收集证据

**当系统有多个组件（API → 服务 → 数据库，CI → 构建 → 部署）：**

**在提出修复之前，添加诊断检测：**

对于每个组件边界：
- 记录什么数据进入组件
- 记录什么数据退出组件
- 验证环境/配置传播
- 检查每层的状态

运行一次收集证据显示**在哪里**断开。
然后分析证据识别失败的组件。
然后调查该特定组件。

#### 5. 跟踪数据流

**当错误在调用栈深处：**

- 坏值在哪里起源？
- 什么用坏值调用了这个函数？
- 继续向上游跟踪直到找到源
- 在源处修复，不在症状处

**行动：** 使用 `search_files` 跟踪引用：

```python
# 查找函数在哪里被调用
search_files("function_name(", path="src/", file_glob="*.py")

# 查找变量在哪里设置
search_files("variable_name\\s*=", path="src/", file_glob="*.py")
```

#### 阶段 1 完成检查清单

- [ ] 错误消息完全阅读和理解
- [ ] 紧密循环命令存在并已至少运行一次
- [ ] 循环是红能力的：它断言用户的确切症状，不是附近的失败
- [ ] 循环是确定性的，或不稳定 bug 有足够高的复现率可以调试
- [ ] 最近更改已识别和审查
- [ ] 证据已收集（日志、状态、数据流）
- [ ] 问题已隔离到特定组件/代码
- [ ] 根因假设可以陈述和测试

**停止：** 在你理解**为什么**它发生之前不要继续到阶段 2。

### 阶段 2：模式分析

**在修复前找到模式：**

#### 0. 最小化复现

一旦循环变红，将复现缩小到仍然变红的最小场景。逐个切割输入、调用者、配置、数据和步骤，每次切割后重新运行循环。只保留对失败起支撑作用的内容。

完成当移除任何剩余元素使循环变绿。最小复现缩小假设空间，通常成为最干净的回归测试。

#### 1. 找到工作示例

- 在同一代码库中定位类似的工作代码
- 什么工作与破碎的类似？

**行动：** 使用 `search_files` 查找可比较的模式：

```python
search_files("similar_pattern", path="src/", file_glob="*.py")
```

#### 2. 与参考比较

- 如果实现模式，完整阅读参考实现
- 不要略读——阅读每一行
- 在应用前完全理解模式

#### 3. 识别差异

- 工作和破碎之间有什么不同？
- 列出每个差异，无论多小
- 不要假设"那不重要"

#### 4. 理解依赖

- 这需要什么其他组件？
- 什么设置、配置、环境？
- 它做什么假设？

### 阶段 3：假设和测试

**科学方法：**

#### 1. 形成排序的可证伪假设

- 在测试任何单个假设之前生成 3-5 个合理假设。
- 按可能性和证伪成本排序。
- 陈述每个假设的预测："如果 X 是原因，那么改变或观察 Y 应该使 Z 发生。"
- 丢弃或锐化任何不做可测试预测的假设。

如果用户在场，在测试前显示排序列表。他们可能有立即重新排序的领域知识。如果用户离开，按你的排序继续。

#### 2. 最小化测试

- 用最小可能的探针测试最高排序的假设。
- 一次改变一个变量。
- 不要同时修复多个东西。
- 当可用时优先调试器/REPL 检查；一个断点胜过十个日志。
- 如果你添加日志，用唯一前缀标记每个临时行，如 `[DEBUG-a4f2]`，使清理成为单次搜索。

#### 3. 继续前验证

- 它工作了吗？→ 阶段 4
- 没有工作？→ 形成新假设
- 不要在上面添加更多修复

#### 4. 当你不知道时

- 说"我不理解 X"
- 不要假装知道
- 请求用户帮助
- 研究更多

### 阶段 4：实现

**修复根因，不是症状：**

#### 1. 创建失败测试用例

- 最简单的可能复现
- 如果可能，自动化测试
- 修复前必须有
- 使用 `test-driven-development` skill

#### 2. 实现单个修复

- 解决识别的根因
- 一次一个更改
- 没有"既然我在这里"的改进
- 没有捆绑重构

#### 3. 验证修复

```bash
# 运行特定回归测试
pytest tests/test_module.py::test_regression -v

# 运行完整套件——没有回归
pytest tests/ -q
```

#### 4. 如果修复不工作——三规则

- **停止。**
- 计数：你尝试了多少修复？
- 如果 < 3：返回阶段 1，用新信息重新分析
- **如果 ≥ 3：停止并质疑架构（下面的步骤 5）**
- 不要尝试修复 #4 没有架构讨论

#### 5. 如果 3+ 修复失败：质疑架构

**指示架构问题的模式：**
- 每个修复在不同地方揭示新的共享状态/耦合
- 修复需要"大规模重构"来实现
- 每个修复在其他地方创建新症状

**停止并质疑基础：**
- 这个模式从根本上合理吗？
- 我们"通过纯粹惯性坚持它"吗？
- 我们应该重构架构 vs 继续修复症状？

**在尝试更多修复前与用户讨论。**

这不是失败的假设——这是错误的架构。

### 红旗——停止并遵循过程

如果你发现自己在想：
- "现在快速修复，以后调查"
- "只是尝试改变 X 看看是否工作"
- "添加多个更改，运行测试"
- "跳过测试，我会手动验证"
- "可能是 X，让我修复它"
- "我没有完全理解但这可能工作"
- "模式说 X 但我会不同地适应它"
- "这是主要问题：[列出修复没有调查]"
- 在跟踪数据流之前提出解决方案
- **"再尝试一次修复"（当已经尝试 2+ 次）**
- **每个修复在不同地方揭示新问题**

**所有这些都意味着：停止。返回阶段 1。**

**如果 3+ 修复失败：** 质疑架构（阶段 4 步骤 5）。

### 常见合理化

| 借口 | 现实 |
|------|------|
| "问题简单，不需要过程" | 简单问题也有根因。过程对简单 bug 很快。 |
| "紧急情况，没有时间过程" | 系统化调试比猜测检查挣扎更快。 |
| "先尝试这个，然后调查" | 第一个修复设置模式。从一开始就做对。 |
| "我会在确认修复工作后写测试" | 未测试的修复不坚持。测试首先证明它。 |
| "同时多个修复节省时间" | 不能隔离什么工作。导致新 bug。 |
| "参考太长，我会适应模式" | 部分理解保证 bug。完整阅读它。 |
| "我看到问题，让我修复它" | 看到症状 ≠ 理解根因。 |
| "再尝试一次修复"（2+ 次失败后） | 3+ 失败 = 架构问题。质疑模式，不要再修复。 |

## 二、Python 调试工具

### 工具选择

| 工具 | 何时 |
|------|------|
| **`breakpoint()` + pdb** | 本地、交互式、最简单。在源中添加 `breakpoint()`，正常运行，在该行获得 REPL。 |
| **`python -m pdb`** | 在没有源编辑的情况下在 pdb 下启动现有脚本。对快速探索有用。 |
| **`debugpy`** | 远程 / 无头 / "附加到已运行进程。" 谈论 DAP，可从终端脚本化，适用于长寿命进程（网关、守护进程、PTY 子进程）。 |

**从 `breakpoint()` 开始。** 它是最便宜的工作。

### pdb 快速参考

在任何 pdb 提示符内（`(Pdb)`）：

| 命令 | 动作 |
|------|------|
| `h` / `h cmd` | 帮助 |
| `n` | 下一行（步过） |
| `s` | 步入 |
| `r` | 从当前函数返回 |
| `c` | 继续 |
| `unt N` | 继续直到行 N |
| `j N` | 跳转到行 N（仅同一函数） |
| `l` / `ll` | 列出当前行周围的源 / 完整函数 |
| `w` | 哪里（堆栈跟踪） |
| `u` / `d` | 在栈中向上/向下移动 |
| `a` | 打印当前函数的参数 |
| `p expr` / `pp expr` | 打印 / 漂亮打印表达式 |
| `display expr` | 每次停止时自动打印表达式 |
| `b file:line` | 设置断点 |
| `b func` | 在函数入口断点 |
| `b file:line, cond` | 条件断点 |
| `cl N` | 清除断点 N |
| `tbreak file:line` | 一次性断点 |
| `!stmt` | 执行任意 Python（包括赋值） |
| `interact` | 在当前作用域中放入完整 Python REPL（Ctrl+D 退出） |
| `q` | 退出 |

`interact` 命令是最强大的——你可以导入任何东西、检查复杂对象、甚至调用改变状态的方法。默认情况下 locals 是只读的；使用 `!x = 42` 从 `(Pdb)` 提示符改变。

### 配方 1：本地断点

最简单。编辑文件：

```python
def compute(x, y):
    result = some_helper(x)
    breakpoint()           # <-- 在这里放入 pdb
    return result + y
```

正常运行代码。你在 `breakpoint()` 行落地，完全访问 locals。

**不要忘记在提交前移除 `breakpoint()`。** 使用 `git diff` 或预提交 grep：
```bash
rg -n 'breakpoint\(\)' --type py
```

### 配方 2：在 pdb 下启动脚本（无源编辑）

```bash
python -m pdb path/to/script.py arg1 arg2
# 在脚本第一行落地
(Pdb) b path/to/script.py:42
(Pdb) c
```

### 配方 3：调试 pytest 测试

hermes 测试运行器和 pytest 都支持这个：

```bash
# 在失败时放入 pdb（或任何引发的异常）：
scripts/run_tests.sh tests/path/to/test_file.py::test_name --pdb

# 在测试开始时放入 pdb：
scripts/run_tests.sh tests/path/to/test_file.py::test_name --trace

# 在跟踪backs中显示 locals 没有 pdb：
scripts/run_tests.sh tests/path/to/test_file.py --showlocals --tb=long
```

注意：`scripts/run_tests.sh` 在捕获的子进程中运行每个测试文件通过 `run_tests_parallel.py`（没有 xdist），所以交互式 pdb 在包装器下不工作。直接运行 pytest 用于 `--pdb`：

```bash
source .venv/bin/activate
python -m pytest tests/foo_test.py::test_bar --pdb
```

这绕过了 hermetic-env 保证——对调试没问题，但在推送前在包装器下重新运行确认。

### 配方 4：任何异常的事后分析

```python
import pdb, sys
try:
    run_the_thing()
except Exception:
    pdb.post_mortem(sys.exc_info()[2])
```

或包装整个脚本：

```bash
python -m pdb -c continue script.py
# 当它崩溃时，pdb 捕获它，你在异常的框架中
```

或在 repl/jupyter 中设置全局钩子：

```python
import sys
def excepthook(etype, value, tb):
    import pdb; pdb.post_mortem(tb)
sys.excepthook = excepthook
```

### 配方 5：用 debugpy 远程调试（附加到运行进程）

对长寿命进程：Hermes 网关、tui_gateway、守护进程、已经行为不端且不能干净重启的进程。

#### 设置

```bash
source /home/bb/hermes-agent/.venv/bin/activate
pip install debugpy
```

#### 模式 A：源编辑——进程在启动时等待调试器

在入口点顶部附近添加（或在你想调试的函数内）：

```python
import debugpy
debugpy.listen(("127.0.0.1", 5678))
print("debugpy listening on 5678, waiting for client...", flush=True)
debugpy.wait_for_client()
debugpy.breakpoint()       # 可选：一旦附加就立即暂停
```

启动进程；它阻塞在 `wait_for_client()`。

#### 模式 B：无源编辑——用 `-m debugpy` 启动

```bash
python -m debugpy --listen 127.0.0.1:5678 --wait-for-client your_script.py arg1
```

模块入口等效：

```bash
python -m debugpy --listen 127.0.0.1:5678 --wait-for-client -m your.module
```

#### 模式 C：附加到已运行进程

需要 PID 和目标环境中预安装的 debugpy：

```bash
python -m debugpy --listen 127.0.0.1:5678 --pid <pid>
# debugpy 将自己注入进程。然后按下面附加客户端。
```

一些内核/安全配置阻止基于 ptrace 的注入（`/proc/sys/kernel/yama/ptrace_scope`）。修复：
```bash
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

#### 从终端连接客户端

最简单的终端端 DAP 客户端是 VS Code CLI 或小脚本。从 Hermes 内部你有两个实际选项：

**选项 1：`debugpy` 自己的 CLI REPL** ——不是官方特性，但是小 DAP 客户端脚本：

```python
# /tmp/dap_client.py
import socket, json, itertools, time, sys

HOST, PORT = "127.0.0.1", 5678
s = socket.create_connection((HOST, PORT))
seq = itertools.count(1)

def send(msg):
    msg["seq"] = next(seq)
    body = json.dumps(msg).encode()
    s.sendall(f"Content-Length: {len(body)}\r\n\r\n".encode() + body)

def recv():
    header = b""
    while b"\r\n\r\n" not in header:
        header += s.recv(1)
    length = int(header.decode().split("Content-Length:")[1].split("\r\n")[0].strip())
    body = b""
    while len(body) < length:
        body += s.recv(length - len(body))
    return json.loads(body)

send({"type": "request", "command": "initialize", "arguments": {"adapterID": "python"}})
print(recv())
send({"type": "request", "command": "attach", "arguments": {}})
print(recv())
send({"type": "request", "command": "setBreakpoints",
      "arguments": {"source": {"path": sys.argv[1]},
                    "breakpoints": [{"line": int(sys.argv[2])}]}})
print(recv())
send({"type": "request", "command": "configurationDone"})
# ... 循环读取事件并发送 continue/stepIn/etc.
```

这对一次性自动化没问题，但作为交互式 UX 很痛苦。

**选项 2：从 VS Code / Cursor / Zed 附加** ——如果用户有一个打开的，他们可以添加 `launch.json`：

```json
{
  "name": "Attach to Hermes",
  "type": "debugpy",
  "request": "attach",
  "connect": { "host": "127.0.0.1", "port": 5678 },
  "justMyCode": false,
  "pathMappings": [
    { "localRoot": "${workspaceFolder}", "remoteRoot": "/home/bb/hermes-agent" }
  ]
}
```

**选项 3：丢弃 DAP，使用 `remote-pdb`** ——通常是你真正想要的从终端代理：

```bash
pip install remote-pdb
```

在你的代码中：
```python
from remote_pdb import set_trace
set_trace(host="127.0.0.1", port=4444)   # 阻塞直到连接
```

然后从终端：
```bash
nc 127.0.0.1 4444
# 你获得 (Pdb) 提示符，就像本地调试一样。
```

`remote-pdb` 是当 `debugpy` 的 DAP 协议过度时最干净的代理友好选择。只有当你真正需要 IDE 集成时才使用 `debugpy`。

### 常见陷阱

1. **在并行/输出捕获运行器下的 pdb 默默地什么都不做。** 你不会看到提示符，测试只是挂起（pytest-xdist 和 `scripts/run_tests.sh` 的捕获每文件子进程都是这样）。直接在单文件上运行 pytest 用于交互式调试。

2. **`breakpoint()` 在 CI / 非 TTY 上下文中挂起进程。** 本地安全；永远不要提交它。添加预提交 grep 作为安全网。

3. **`PYTHONBREAKPOINT=0`** 禁用所有 `breakpoint()` 调用。如果你的断点没有命中，检查环境：
   ```bash
   echo $PYTHONBREAKPOINT
   ```

4. **`debugpy.listen` 只有在你也调用 `wait_for_client()` 时才阻塞。** 没有它，执行继续，你的第一个断点可能在客户端附加前触发。

5. **附加到 PID 在强化内核上失败。** `ptrace_scope=1`（Ubuntu 默认）只允许同用户 ptrace 子进程。变通：`echo 0 > /proc/sys/kernel/yama/ptrace_scope`（需要 root）或从一开始在 `debugpy` 下启动。

6. **线程。** `pdb` 只调试当前线程。对多线程代码，使用 `debugpy`（线程感知 DAP）或每线程设置 `threading.settrace()`。

7. **asyncio。** `pdb` 在协程中工作，但 pdb 内的 `await` 需要 Python 3.13+ 或旧版本上从 `interact` 模式的 `await`。对 3.11/3.12，使用 `asyncio.run_coroutine_threadsafe` 技巧或通过 `asyncio.ensure_future` 的 `!stmt` 等待。

8. **`scripts/run_tests.sh` 剥离凭据并设置 `HOME=<tmpdir>`。** 如果你的 bug 依赖用户配置或真实 API keys，它不会在包装器下复现。先用原始 `pytest` 调试复现，然后在包装器下重新确认。

9. **分叉 / 多进程。** pdb 不跟随分叉。每个子进程需要自己的 `breakpoint()` 或 `set_trace()`。对 Hermes 子代理，一次调试一个进程。

## 三、Node.js 调试工具

### 工具选择

- **`node inspect`** ——内置、零安装、CLI REPL。对快速探索最好。
- **`ndb` / CDP 通过 `chrome-remote-interface`** ——可从 Node/Python 脚本化；对当你想自动化许多断点、跨运行收集状态或从代理循环非交互式调试时最好。

**优先 `node inspect` 首先。** 它总是可用的，REPL 很快。

### 快速参考：`node inspect` REPL

在第一行暂停启动：

```bash
node inspect path/to/script.js
# 或用 tsx
node --inspect-brk $(which tsx) path/to/script.ts
```

`debug>` 提示符接受：

| 命令 | 动作 |
|------|------|
| `c` 或 `cont` | 继续 |
| `n` 或 `next` | 步过 |
| `s` 或 `step` | 步入 |
| `o` 或 `out` | 步出 |
| `pause` | 暂停运行代码 |
| `sb('file.js', 42)` | 在 file.js 行 42 设置断点 |
| `sb(42)` | 在当前文件行 42 设置断点 |
| `sb('functionName')` | 函数调用时断点 |
| `cb('file.js', 42)` | 清除断点 |
| `breakpoints` | 列出所有断点 |
| `bt` | 回溯（调用栈） |
| `list(5)` | 显示当前位置周围 5 行源 |
| `watch('expr')` | 每次暂停时评估表达式 |
| `watchers` | 显示监视的表达式 |
| `repl` | 在当前作用域中放入 REPL（Ctrl+C 退出 REPL） |
| `exec expr` | 评估表达式一次 |
| `restart` | 重启脚本 |
| `kill` | 杀死脚本 |
| `.exit` | 退出调试器 |

**在 `repl` 子模式中：** 输入任何 JS 表达式，包括访问 locals/closure 变量。`Ctrl+C` 退出回到 `debug>`。

### 附加到运行进程

当进程已经在运行（例如长寿命开发服务器或 TUI 网关）：

```bash
# 1. 发送 SIGUSR1 在现有进程上启用检查器
kill -SIGUSR1 <pid>
# Node 打印：Debugger listening on ws://127.0.0.1:9229/<uuid>

# 2. 附加调试器 CLI
node inspect -p <pid>
# 或通过 URL
node inspect ws://127.0.0.1:9229/<uuid>
```

从一开始启动带检查器的进程：

```bash
node --inspect script.js           # 在 127.0.0.1:9229 监听，继续运行
node --inspect-brk script.js       # 监听并在第一行暂停
node --inspect=0.0.0.0:9230 script.js   # 自定义 host:port
```

对 TypeScript 通过 tsx：

```bash
node --inspect-brk --import tsx script.ts
# 或旧 tsx
node --inspect-brk -r tsx/cjs script.ts
```

### 程序化 CDP（从终端脚本化）

当你想自动化——设置许多断点、捕获作用域状态、脚本化复现——使用 `chrome-remote-interface`：

```bash
npm i -g chrome-remote-interface        # 或项目本地
# 启动你的目标：
node --inspect-brk=9229 target.js &
```

驱动脚本（保存为 `/tmp/cdp-debug.js`）：

```javascript
const CDP = require('chrome-remote-interface');

(async () => {
  const client = await CDP({ port: 9229 });
  const { Debugger, Runtime } = client;

  Debugger.paused(async ({ callFrames, reason }) => {
    const top = callFrames[0];
    console.log(`PAUSED: ${reason} @ ${top.url}:${top.location.lineNumber + 1}`);

    // 遍历作用域查找 locals
    for (const scope of top.scopeChain) {
      if (scope.type === 'local' || scope.type === 'closure') {
        const { result } = await Runtime.getProperties({
          objectId: scope.object.objectId,
          ownProperties: true,
        });
        for (const p of result) {
          console.log(`  ${scope.type}.${p.name} =`, p.value?.value ?? p.value?.description);
        }
      }
    }

    // 在暂停框架中评估表达式
    const { result } = await Debugger.evaluateOnCallFrame({
      callFrameId: top.callFrameId,
      expression: 'typeof state !== "undefined" ? JSON.stringify(state) : "n/a"',
    });
    console.log('state =', result.value ?? result.description);

    await Debugger.resume();
  });

  await Runtime.enable();
  await Debugger.enable();

  // 通过 URL 正则 + 行设置断点
  await Debugger.setBreakpointByUrl({
    urlRegex: '.*app\\.tsx$',
    lineNumber: 119,       // 0-索引
    columnNumber: 0,
  });

  await Runtime.runIfWaitingForDebugger();
})();
```

运行它：

```bash
node /tmp/cdp-debug.js
```

Hermes 特定注意：`chrome-remote-interface` 不在 `ui-tui/package.json` 中。如果你不想弄脏项目，安装到一次性位置：

```bash
mkdir -p /tmp/cdp-tools && cd /tmp/cdp-tools && npm i chrome-remote-interface
NODE_PATH=/tmp/cdp-tools/node_modules node /tmp/cdp-debug.js
```

### 调试 Hermes ui-tui

TUI 构建 Ink + tsx。两个常见场景：

#### 在开发下调试单个 Ink 组件

`ui-tui/package.json` 有 `npm run dev`（tsx --watch）。通过直接运行 tsx 添加 `--inspect-brk`：

```bash
cd /home/bb/hermes-agent/ui-tui
npm run build    # 产生 dist/ 一次，使首次加载不需要转译
node --inspect-brk dist/entry.js
# 在另一个终端：
node inspect -p <node pid>
```

然后在 `debug>` 内：

```
sb('dist/app.js', 220)     # 或可疑渲染在哪里
cont
```

当它暂停时，`repl` → 检查 `props`、状态 refs、`useInput` 处理器值等。

#### 调试运行的 `hermes --tui`

TUI 从 Python CLI 生成 Node。最简单路径：

```bash
# 1. 启动 TUI
hermes --tui &
TUI_PID=$(pgrep -f 'ui-tui/dist/entry' | head -1)

# 2. 在该 Node PID 上启用检查器
kill -SIGUSR1 "$TUI_PID"

# 3. 查找 WS URL
curl -s http://127.0.0.1:9229/json/list | jq -r '.[0].webSocketDebuggerUrl'

# 4. 附加
node inspect ws://127.0.0.1:9229/<uuid>
```

与 TUI 交互（在其窗口中输入）继续推进执行；你的调试器可以在任何 `sb(...)` 的断点处暂停它。

#### 调试 `_SlashWorker` / PTY 子进程

那些是 Python，不是 Node——对它们使用 `python-debugpy` skill。只有 Node 部分（Ink UI、tui_gateway 客户端、`ui-tui/` 下 tsx-run 测试）使用这个 skill。

### 在调试器下运行 Vitest 测试

```bash
cd /home/bb/hermes-agent/ui-tui
# 在入口暂停运行单个测试文件
node --inspect-brk ./node_modules/vitest/vitest.mjs run --no-file-parallelism src/app/foo.test.tsx
```

在另一个终端：`node inspect -p <pid>`，然后 `sb('src/app/foo.tsx', 42)`，`cont`。

使用 `--no-file-parallelism`（vitest）或 `--runInBand`（jest）使只有一个 worker——调试池很痛苦。

### 堆快照和 CPU 配置文件（非交互式）

从上面的 CDP 驱动，交换 Debugger 为 `HeapProfiler` / `Profiler`：

```javascript
// 5 秒 CPU 配置文件
await client.Profiler.enable();
await client.Profiler.start();
await new Promise(r => setTimeout(r, 5000));
const { profile } = await client.Profiler.stop();
require('fs').writeFileSync('/tmp/cpu.cpuprofile', JSON.stringify(profile));
// 在 Chrome DevTools → Performance 标签中打开 /tmp/cpu.cpuprofile
```

```javascript
// 堆快照
await client.HeapProfiler.enable();
const chunks = [];
client.HeapProfiler.addHeapSnapshotChunk(({ chunk }) => chunks.push(chunk));
await client.HeapProfiler.takeHeapSnapshot({ reportProgress: false });
require('fs').writeFileSync('/tmp/heap.heapsnapshot', chunks.join(''));
```

### 常见陷阱

1. **TS 源中错误的行号。** 断点命中发出的 JS，不是 `.ts`。要么（a）在构建的 `dist/*.js` 中断点，或（b）启用 sourcemaps（`node --enable-source-maps`）并使用 `sb('src/app.tsx', N)` ——但只有与跟随 sourcemaps 的 CDP 客户端。`node inspect` CLI 不。

2. **`--inspect` vs `--inspect-brk`。** `--inspect` 启动检查器但不暂停；如果你附加太晚，你的脚本会跑过你的第一个断点。当你需要在任何代码运行前设置断点时使用 `--inspect-brk`。

3. **端口冲突。** 默认是 `9229`。如果多个 Node 进程在检查，传递 `--inspect=0`（随机端口）并从 `/json/list` 读取实际 URL：
   ```bash
   curl -s http://127.0.0.1:9229/json/list   # 列出主机上所有可检查目标
   ```

4. **子进程。** 父进程上的 `--inspect` 不检查其子进程。使用 `NODE_OPTIONS='--inspect-brk' node parent.js` 传播到每个子进程；注意它们都需要唯一端口（当 `NODE_OPTIONS='--inspect'` 被继承时 Node 自动递增）。

5. **后台杀死。** 如果你在目标暂停时从 `node inspect` 中 `Ctrl+C`，目标保持暂停。要么先 `cont`，或明确 `kill` 目标。

6. **通过代理终端运行 `node inspect`。** 它是 PTY 友好的 REPL。在 Hermes 中，用 `terminal(pty=true)` 或 `background=true` + `process(action='submit', data='...')` 启动它。非 PTY 前台模式对一次性命令工作，但对交互式步进不工作。

7. **安全。** `--inspect=0.0.0.0:9229` 暴露任意代码执行。总是绑定到 `127.0.0.1`（默认），除非你有隔离网络。

## 四、常见陷阱汇总

### 方法论
- **不要跳过阶段 1** ——没有根因调查不尝试修复
- **不要同时修复多个东西** ——不能隔离什么工作
- **3+ 修复失败** ——质疑架构，不要继续修复
- **不要合理化跳过过程** ——系统化总是赢

### Python 调试
- **pdb 在并行运行器下不工作** ——直接运行 pytest
- **`breakpoint()` 在 CI 中挂起** ——永远不要提交它
- **`PYTHONBREAKPOINT=0` 禁用断点** ——检查环境
- **debugpy 需要 `wait_for_client()`** ——否则执行继续
- **线程需要特殊处理** ——使用 debugpy 或每线程 settrace

### Node.js 调试
- **TS 源行号错误** ——在 dist/*.js 中断点或启用 sourcemaps
- **`--inspect` vs `--inspect-brk`** ——brk 在第一行暂停
- **端口冲突** ——使用 `--inspect=0` 随机端口
- **子进程不继承检查** ——使用 NODE_OPTIONS 传播
- **Ctrl+C 保持目标暂停** ——先 cont 或 kill

## 五、参考资源

- systematic-debugging：4 阶段根因调试方法论
- python-debugpy：Python 调试工具（pdb, debugpy, remote-pdb）
- node-inspect-debugger：Node.js 调试工具（node inspect, CDP）
- test-driven-development：TDD 工作流（修复 bug 时）
