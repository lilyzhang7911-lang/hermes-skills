---
name: macos-system-diagnosis
description: macOS 系统排错通用方法论。覆盖进程崩溃、端口冲突、VPN 干扰、僵尸任务、服务健康检查分层。基于 Hindsight 真实案例沉淀，可泛化到任何本地服务/守护进程的异常排查。
rational: true
version: 1.0.0
author: Hermes Agent + User
license: MIT
platforms: [macos]
metadata:
  hermes:
    tags: [macos, troubleshooting, crash, diagnosis, system, ops, devops]
---

# macOS 系统排错 Playbook

排查 macOS 上本地服务/守护进程的崩溃、性能退化、连接失败等异常。基于 Hindsight 崩溃/CPU 满/consolidation 失败的真实案例沉淀，方法论可泛化到任何本地服务。

## When to Use

遇到以下任一情况时使用：

- 服务进程频繁崩溃（`Python-*.ips`、`AppName-*.ips` 报告）
- CPU 冲满 100% 且停掉某服务才降温
- 本地 LLM/服务一直"忙碌"但无实际产出
- 服务日志反复出现 `Connection error`、`Timeout`、`STUCK`
- 端口被占用/堵塞，服务起不来或响应异常
- 僵尸任务/操作堆积在队列中（`payload_null`、`PENDING` 永不过去）
- VPN/代理导致本地端口连通性异常

## 核心心智模型：症状 ≠ 根因

一个现象背后常有多个叠加的根因。诊断必须分层，先定位"崩溃性质"，再找"触发入口"，再看"放大因素"。

```
症状(CPU 100%) ← 崩溃-重启循环(KeepAlive) ← MPS 段错误(崩溃) ← 嵌入走 MPS
                 ↖ 放大因素：Lantern VPN 堵塞 1234 → worker 无限重试
                            ↑ retain 是高频触发入口
```

**关键原则：** 不要看到 CPU 100% 就去 top 看哪个进程吃资源；先问：为什么这个进程会高频循环？

---

## 一、通用排查流程（分层，严格按序）

### 第 0 步：确认症状与时间线

```bash
# 崩溃报告目录（按时间倒序）
ls -lt ~/Library/Logs/DiagnosticReports/*.ips | head

# 崩溃时间线（判断是否"崩溃-重启循环"）
ls ~/Library/Logs/DiagnosticReports/*.ips | \
  sed -E 's/.*-([0-9]{8})-([0-9]{6})\..*/\1 \2/' | sort | head -30

# 若 1 小时内 >10 次崩溃且间隔 30-60 秒 → KeepAlive/自动重启机制在循环
```

**同时记录：**
- 症状首次出现时间
- 最近一次系统/软件更新
- 最近一次配置变更
- 是否开启了 VPN/代理

### 第 1 步：定位崩溃性质（读 .ips 崩溃栈）

崩溃报告是 JSON，第二行是详情。**必须解析 faulting thread 的调用栈**：

```bash
python3 -c "
import json, sys
h = open(sys.argv[1]).read().split('\n', 1)
body = json.loads(h[1])
ft = body['faultingThread']
imgs = body['usedImages']
print('EXC:', body['exception'].get('type'), body['exception'].get('signal'))
for fr in body['threads'][ft]['frames'][:12]:
    img = imgs[fr.get('imageIndex')].get('name', '?') if fr.get('imageIndex') is not None else '?'
    print(f'  [{img}]', fr.get('symbol', ''))
" /path/to/*.ips
```

**关键判读：**
- 栈里有 `libtorch_cpu.dylib` + `at::native::mps::*` + `copy_cast_kernel_mps` → **PyTorch MPS 段错误**
- `EXC_BAD_ACCESS` / `SIGSEGV` / `SIGBUS` → 原生层内存错误（C 扩展崩溃），**不是** Python 可捕获异常
- 栈全是 Python 帧 → 应用层异常，查代码/配置
- 栈里有 `libsystem_platform.dylib` / `libsystem_kernel.dylib` → 系统调用层面，查权限/资源

**通用规则：原生层崩溃 ≠ 代码 bug，通常是环境/依赖/配置问题。**

### 第 2 步：服务/网络健康检查

```bash
# 1. 服务自身 health 端点（若有）
curl -s http://localhost:PORT/health

# 2. 端口监听状态
lsof -i :PORT | head -20

# 3. 进程状态
ps aux | grep SERVICE_NAME | grep -v grep
# 或更精确
pgrep -f "SERVICE_PATTERN" && ps -p $(pgrep -f "SERVICE_PATTERN") -o pid,pcpu,pmem,etime

# 4. 网络连通性（区分"服务没起"和"服务起了但连不上"）
# 本机回环
curl -s --max-time 5 http://localhost:PORT/health
# 外部可达性
curl -s --max-time 5 http://127.0.0.1:PORT/health
```

**关键陷阱：health 通过 ≠ 系统正常。** 很多服务的 health 只查数据库/存储，不查下游依赖（LLM、外部 API）。必须单独验证关键链路。

### 第 3 步：日志与证据收集

```bash
# 服务日志（若有标准日志位置）
tail -100 ~/Library/Logs/SERVICE_NAME/*.log

# 系统日志（通用）
log show --predicate 'process == "SERVICE_NAME"' --last 1h --style syslog | tail -50

# 崩溃相关系统日志
log show --predicate 'eventMessage CONTAINS "SERVICE_NAME"' --last 1h | tail -50

# 关键词搜索
grep -i "error\|exception\|fatal\|panic\|crash\|timeout\|connection" /path/to/log
```

**收集原则：** 先收集，再分析。把日志、进程状态、端口状态、崩溃报告**一次性 dump** 到临时目录，避免反复切换上下文。

### 第 4 步：根因模式匹配

见下方「通用模式库」。

### 第 5 步：验证（必须，别停在"看起来好了"）

```bash
# 1. 确认服务连接验证成功
grep "Connection verified\|Connection error" /path/to/log | tail -2

# 2. 确认无新崩溃
find ~/Library/Logs/DiagnosticReports -name "*.ips" -newermt "今天" | wc -l

# 3. 进程稳定、CPU/内存正常
ps aux | grep SERVICE_NAME | grep -v grep | awk '{print $2, $3, $4}'

# 4. 手动触发一次真实操作验证
# （见具体服务的验证命令）
```

**稳定性验证铁律：** 别一步全开自动/生产模式。先手动触发一次真实操作，确认不崩溃，再推进。

---

## 二、通用模式库

### 模式 1：原生库段错误（MPS / CUDA / 其他）

**症状：** 服务在特定操作时 SIGSEGV/SIGBUS，崩溃报告指向原生库（`libtorch`、`libcuda`、`libomp` 等）

**根因：** 本地硬件加速路径（MPS/CUDA/OpenCL）与特定算子/数据类型不兼容

**通用修复思路：**
1. 查该库是否有"禁用硬件加速"的环境变量
2. 若有，强制走 CPU/回退路径
3. 验证：相同操作不再崩溃

**Hindsight 实例：**
```bash
# PyTorch MPS 强制 CPU
export HINDSIGHT_API_EMBEDDINGS_LOCAL_FORCE_CPU=1
export HINDSIGHT_API_RERANKER_LOCAL_FORCE_CPU=1
# 不要用 PYTORCH_ENABLE_MPS_FALLBACK=1（只回退不支持的算子，救不了 MPS 自身崩溃）
```

### 模式 2：VPN/代理堵塞本地端口

**症状：** 本地服务 health 正常，但实际请求超时；`/v1/models` 通但 `/v1/chat` 失败

**根因：** VPN "代理所有内容"向本地推理/服务端口发连接，造成堵塞

**通用修复思路：**
1. 确认 VPN 进程：`lsof -i :PORT -c VPN_PROCESS | wc -l`
2. 若 >50 条连接指向同一本地端口 → VPN 嫌疑
3. 退出 VPN 或配置绕过规则
4. 重启服务（新进程会重新建立连接）

**Hindsight 实例：**
```bash
lsof -i :1234 -c Lantern | wc -l  # >100 = Lantern 堵塞
# 完全退出 Lantern，重启 Hindsight
```

### 模式 3：崩溃-重启循环（自动恢复机制变成放大器）

**症状：** 服务频繁崩溃，但因为有 KeepAlive/auto-restart 机制，看起来"一直在运行"

**根因：** 自动恢复掩盖了真实崩溃。每次重启后服务重新执行相同操作 → 再次崩溃

**通用修复思路：**
1. 先确认是循环：`find ~/Library/Logs/DiagnosticReports -name "*.ips" -newermt "1小时前" | wc -l`
2. 若 >10 次/小时 → 循环
3. 修根因，而不是禁用自动重启
4. 临时止血：暂停自动重启，人工确认每次失败原因

### 模式 4：僵尸任务/操作（数据丢失后的空壳）

**症状：** 队列中任务永远 `PENDING`，日志出现 `payload_null`、`claimable=0`

**根因：** 崩溃时部分任务的 payload 丢失，任务壳还在但内容已空

**通用修复思路：**
1. 区分"卡住"和"僵尸"：
   - 卡住：有 payload，可能恢复
   - 僵尸：payload 为空，内容已丢
2. 有 payload 的卡住任务 → 等恢复或手动重试
3. payload 为空的僵尸任务 → 安全清理

**Hindsight 实例：**
```sql
-- 先确认哪些是僵尸
SELECT operation_id, status, task_payload IS NULL AS is_zombie
FROM async_operations
WHERE status IN ('pending', 'processing');

-- 只删僵尸（有 payload 的别删！）
DELETE FROM async_operations
WHERE operation_id IN (...) AND task_payload IS NULL;
```

### 模式 5：健康检查假阳性

**症状：** 服务 `/health` 返回 200，但实际功能不可用

**根因：** health 端点只检查了部分依赖（通常是存储层），跳过了关键外部依赖（LLM、API、网络）

**通用修复思路：**
1. 区分"存储健康"和"功能健康"
2. 手动验证关键链路：
   - 存储层：`/health` 或数据库连接
   - 计算层：本地模型/推理引擎
   - 网络层：外部 API/上游服务
3. 建立多层健康检查，不依赖单一端点

### 模式 6：端口冲突

**症状：** 服务启动失败，提示 "Address already in use"

**通用修复思路：**
```bash
# 1. 谁占了端口
lsof -i :PORT
# 或
lsof -i tcp:PORT

# 2. 杀死占用进程
kill -9 PID

# 3. 若杀不掉，查是否有系统服务占用
sudo lsof -i :PORT

# 4. 临时规避：改服务端口
```

---

## 三、排查决策树

```
服务异常
├── 服务没启动
│   ├── 端口被占用 → 模式 6
│   ├── 权限不足 → 查 TCC/权限设置
│   └── 配置错误 → 查配置文件语法
├── 服务启动了但不工作
│   ├── health 正常但功能异常 → 模式 5（健康检查假阳性）
│   ├── 连接超时 → 模式 2（VPN/代理）或模式 6（端口冲突）
│   └── 日志有错误 → 收集日志 → 模式匹配
├── 服务频繁崩溃
│   ├── 原生层崩溃（.ips） → 模式 1
│   ├── 崩溃-重启循环 → 模式 3
│   └── 应用层异常 → 查代码/依赖/配置
└── 性能异常（CPU/内存高）
    ├── 有崩溃循环 → 模式 3
    ├── 无崩溃但高负载 → 查进程内部状态（队列积压、线程数）
    └── 僵尸任务堆积 → 模式 4
```

---

## 四、工具速查

| 目的 | 命令 |
|------|------|
| 查看崩溃报告 | `ls -lt ~/Library/Logs/DiagnosticReports/*.ips` |
| 解析崩溃栈 | `python3 -c "..." /path/to/*.ips` |
| 端口监听 | `lsof -i :PORT` |
| 进程状态 | `ps aux \| grep NAME` / `pgrep -f PATTERN` |
| 网络连通性 | `curl -s --max-time 5 http://localhost:PORT/health` |
| VPN 连接数 | `lsof -i :PORT -c VPN_NAME \| wc -l` |
| 系统日志 | `log show --predicate 'process == "NAME"' --last 1h` |
| 僵尸任务（通用） | 查任务队列中 payload 为空的条目 |
| 强制重启服务 | `kill -HUP PID` 或 `brew services restart NAME` |

---

## 五、稳定性验证（修复后必须做）

**铁律：** 别一步全开自动/生产模式。先手动触发一次真实操作，确认不崩溃，再推进。

```bash
# 1. 确认服务连接验证成功
grep "Connection verified\|Connection error" /path/to/log | tail -2

# 2. 确认无新崩溃
find ~/Library/Logs/DiagnosticReports -name "*.ips" -newermt "今天" | wc -l

# 3. 进程稳定、资源正常
ps aux | grep SERVICE_NAME | grep -v grep | awk '{print $2, $3, $4}'

# 4. 手动触发一次真实操作验证
# （根据具体服务调整）

# 5. 观察期（至少 10-15 分钟）
watch -n 5 'ps aux | grep SERVICE_NAME | grep -v grep'
```

---

## 六、Pitfalls 与教训

- **health 通过 ≠ 系统正常**：health 只查部分依赖，必须手动验证关键链路
- **"卡住" ≠ "僵尸"**：卡住可能恢复；僵尸（payload_null）内容已丢才可删
- **自动重启是双刃剑**：崩溃自动重启掩盖了真实问题，修根因而不是依赖它
- **CPU 100% 的元凶往往是循环**：不是某个进程"吃资源"，而是某个机制让进程高频重复执行
- **VPN 的影响是全局的**：不只是浏览器，所有本地端口都可能被堵塞
- **原生层崩溃 ≠ 代码 bug**：通常是环境/依赖/配置问题，不要陷入代码审查
- **"快速修复"往往引入新问题**：先建立 tight loop，再系统性修复

---

## 七、案例：Hindsight 崩溃-CPU 100% 完整排查

### 症状
- CPU 100%，停掉 Hindsight 才降温
- LM Studio 模型一直"忙碌"但无产出
- `~/Library/Logs/DiagnosticReports/` 下频繁出现 `Python-*.ips`

### 根因链
```
症状(CPU 100%)
  ← 崩溃-重启循环（KeepAlive 每 30-60 秒重启一次）
    ← MPS 段错误（PyTorch 嵌入阶段 SIGSEGV）
      ← Apple Silicon 上 MPS 路径不稳定
    ← 放大因素：Lantern VPN 堵塞 1234 端口
      ← worker 无限重试 LLM → LM Studio 高 CPU 无产出
```

### 修复
1. 强制嵌入/重排走 CPU（环境变量）
2. 完全退出 Lantern
3. 清理僵尸操作（`payload_null` 的 pending 任务）
4. 手动触发 consolidation 验证
5. 确认无新崩溃后，开启自动 consolidation

### 验证
- 日志出现 `forcing CPU mode`
- `Connection verified` 成功
- 1 小时内 0 新崩溃
- CPU 恢复正常水平

---

## 八、相关 Skill

- `hindsight-crash-diagnosis`：Hindsight 专属排查 Playbook（本 skill 的母本）
- `hindsight-deployment`：Hindsight 部署与配置
- `systematic-debugging`：通用 4 阶段根因调试方法论
- `macos-automation`：macOS TCC/权限/感知失败排查
- `macos-proxy-troubleshooting`：VPN/代理连通性问题
- `diagnosing-bugs`：通用故障诊断循环

---

## 九、使用建议

1. **先按流程走，再按模式查**：第 0-5 步是通用流程，模式库是常见场景速查
2. **收集证据再分析**：不要边查边修，先 dump 完整证据
3. **区分"卡住"和"崩溃"**：这是两类完全不同的根因
4. **修根因，不是修症状**：崩溃循环的根源是 MPS 段错误，不是 KeepAlive
5. **验证必须手动触发真实操作**：health 通过不等于功能正常
