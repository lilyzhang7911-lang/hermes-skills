---
name: hindsight-crash-diagnosis
description: 排查Hindsight崩溃/CPU满/consolidation失败：MPS段错误、VPN堵塞、崩溃循环、僵尸操作。
rational: true
version: 1.0.0
author: Hermes Agent + User
license: MIT
platforms: [macos]
metadata:
  hermes:
    tags: [hindsight, crash, diagnosis, mps, troubleshooting, macos]
---

# Hindsight 崩溃与性能排查 Playbook

排查 macOS 上 Hindsight 本地记忆服务的崩溃/性能故障。基于 2026-08-08/09 真实排查案例（MPS 段错误崩溃循环 + Lantern 堵塞 + CPU 100%）沉淀。

## When to Use

遇到以下任一情况时使用本 skill 排查 Hindsight 崩溃/性能问题：

- Hindsight 崩溃报告（`Python-*.ips`）频繁出现
- CPU 冲满 100% 且停掉 Hindsight 才降温
- LM Studio 模型一直"忙碌"但无实际产出
- consolidation / retain 反复 `APIConnectionError` 或 `Connection error` 失败
- 日志出现 `payload_null`、`STUCK`、`Failed to claim`

## 核心心智模型：症状 ≠ 根因

一个现象背后常有多个叠加的根因。诊断必须分层，先定位"崩溃性质"，再找"触发入口"，再看"放大因素"。

```
症状(CPU 100%) ← 崩溃-重启循环(KeepAlive) ← MPS 段错误(崩溃) ← 嵌入走 MPS
                 ↖ 放大因素：Lantern VPN 堵塞 1234 → worker 无限重试
                            ↑ retain 是高频触发入口
```

## 排查流程（分层，严格按序）

### 第 0 步：确认症状与时间线
```bash
# 崩溃报告目录（按时间倒序）
ls -lt ~/Library/Logs/DiagnosticReports/Python-*.ips | head
# 崩溃时间线（判断是否"崩溃-重启循环"）
ls Python-*.ips | sed -E 's/.*-([0-9]{8})-([0-9]{6})\..*/\1 \2/' | sort | head -30
# 若 1 小时内 >10 次崩溃且间隔 30-60 秒 → KeepAlive 崩溃-重启循环
```

### 第 1 步：定位崩溃性质（读 .ips 崩溃栈）
崩溃报告是 JSON，第二行是详情。**必须解析 faulting thread 的调用栈**：
```bash
python3 -c "
import json,sys
h=open(sys.argv[1]).read().split('\n',1)
body=json.loads(h[1])
ft=body['faultingThread']; imgs=body['usedImages']
print('EXC:', body['exception'].get('type'), body['exception'].get('signal'))
for fr in body['threads'][ft]['frames'][:12]:
    img=imgs[fr.get('imageIndex')].get('name','?') if fr.get('imageIndex') is not None else '?'
    print(f'  [{img}]', fr.get('symbol',''))
" /path/to/Python-*.ips
```
**关键判读**：
- 栈里有 `libtorch_cpu.dylib` + `at::native::mps::*` + `copy_cast_kernel_mps` / `exec_unary_kernel` → **PyTorch MPS 段错误**
- `EXC_BAD_ACCESS` / `SIGSEGV` / `SIGBUS` → 原生层内存错误（C 扩展崩溃），**不是** Python 可捕获异常。别查代码逻辑，是原生库。

### 第 2 步：服务/网络健康检查
```bash
curl -s http://localhost:8888/health          # 注意：health 只查数据库，不查 LLM！
# LLM 连通性（真正的关键）
curl -s --max-time 8 -X POST http://localhost:1234/v1/chat/completions -H "Content-Type: application/json" -d '{"model":"MODEL","messages":[{"role":"user","content":"hi"}],"max_tokens":5}'
# 若 /v1/models 通但 /v1/chat 超时/Connection error → 疑似 VPN 堵塞
lsof -i :1234 -c Lantern | wc -l              # >100 = Lantern 堵塞
lsof -i :1234 | wc -l                          # 总数
```

### 第 3 步：根因模式匹配（见下方模式库）

### 第 4 步：修复

### 第 5 步：验证（必须，别停在"看起来好了"）
```bash
# 1. 重启后确认 LLM 连接验证成功
grep "Connection verified\|Connection error" ~/Library/Logs/Hindsight/hindsight.log | tail -2
# 2. 确认嵌入走 CPU
grep "forcing CPU mode" ~/Library/Logs/Hindsight/hindsight.log | tail -2
# 3. 确认无新崩溃
find ~/Library/Logs/DiagnosticReports -name "Python-*.ips" -newermt "今天" | wc -l   # 应为 0-1（重启期间）
# 4. 进程稳定、CPU 正常
ps aux | grep hindsight-api | grep -v grep | awk '{print $2, $3}'
# 5. 手动触发一次真实操作验证（见"稳定性验证"）
```

## 根因模式库

### 模式 1：PyTorch MPS 段错误（核心）
- **症状**：consolidation/retain 的嵌入阶段 SIGSEGV，KeepAlive 反复重启，CPU 100%
- **根因**：Apple Silicon 上 PyTorch 自动检测 MPS 并用它跑嵌入/重排，`tensor.to('mps')` → `copy_cast_kernel_mps` 原生崩溃
- **修复**：plist 加两个官方开关强制 CPU：
  - `HINDSIGHT_API_EMBEDDINGS_LOCAL_FORCE_CPU=1`
  - `HINDSIGHT_API_RERANKER_LOCAL_FORCE_CPU=1`
  - **不要**用 `PYTORCH_ENABLE_MPS_FALLBACK=1`（只回退不支持的算子，救不了 MPS 自身的崩溃，反让 MPS 路径继续被选中）
- **性能影响≈0**：bge-small 仅 33M 参数，CPU 嵌入几十毫秒/条

### 模式 2：Lantern VPN 堵塞 LM Studio 端口
- **症状**：`/v1/models` 通但 `/v1/chat` Connection error；`lsof -i :1234 -c Lantern` >100
- **根因**：Lantern "代理所有内容"向所有本地端口发连接，堵塞 1234 推理端口
- **修复**：退出 Lantern（关闭"代理所有内容"可能不够，实测需完全退出）
- **陷阱**：堵塞期间 Hindsight 启动 `Verifying connection` 失败，但 `/health` 仍 healthy（只查库不查 LLM）→ 服务起了但 LLM 不可用，所有 retain/consol 失败

### 模式 3：崩溃-重启循环（CPU 100% 的直接推手）
- **机制**：`KeepAlive=true` 崩溃立即重启 → worker 又处理嵌入 → 又崩 → 又重启。每 30-60 秒一次，CPU 满载
- **识别**：崩溃时间线 1 小时内 >10 次、间隔短
- **修复**：修掉崩溃源头（模式 1）。止赎循环 = 断源头，不是改 KeepAlive
- **陷阱**：worker 崩溃/堵塞期间反复无效重试 LLM，导致 LM Studio 模型一直"忙碌"高 CPU 但无产出——堵塞解决、worker 空闲后模型才回归空闲

### 模式 4：僵尸操作（payload_null）
- **症状**：worker 日志 `[PENDING_BREAKDOWN] batch_retain: total=N claimable=0 payload_null=N`
- **根因**：崩溃时部分 batch_retain 的 payload 丢失（`task_payload IS NULL`），永远无法 claim/处理
- **识别**：`task_payload IS NULL` 的 pending 操作。**有 payload 的 retain 是真实任务，删了会丢内容**——只有 payload_null 才可安全删
- **修复**：直接 SQL 删除（cancel 工具只支持 pending 且 payload_null 无法 cancel）：
  ```bash
  PSQL="/Users/sunwenning/.pg0/installation/18.1.0/bin/psql -h 127.0.0.1 -p 5433 -U hindsight -d hindsight"
  # 凭证在 ~/.pg0/instances/hindsight/instance.json（默认 hindsight/hindsight）
  # 1. 先备份
  $PSQL -c "\COPY (SELECT * FROM async_operations WHERE operation_id IN (...)) TO /tmp/backup.sql"
  # 2. 再删（async_operations 无外键引用，安全，不影响 memory_units/chunks/entities）
  $PSQL -c "DELETE FROM async_operations WHERE operation_id IN (...) RETURNING operation_id, status;"
  # 3. 验证队列清空
  $PSQL -c "SELECT status, count(*) FROM async_operations WHERE status IN ('pending','processing') GROUP BY status;"
  ```

## 稳定性验证（修复后必须做，分步推进）

**教训**：别一步全开自动。先手动触发一次真实操作，确认不崩溃，再开自动。

```bash
# 1. 查待处理量
export PGPASSWORD=hindsight
$PSQL -c "SELECT count(*) FILTER (WHERE consolidated_at IS NULL AND consolidation_failed_at IS NULL) FROM memory_units;"
# 2. 手动触发 consolidation（REST 端点）
curl -s -X POST http://localhost:8888/v1/default/banks/hermes/consolidate -H "Content-Type: application/json" -d '{}'
# 3. 挂后台监控直到 completed（轮询 async_operations 状态）
#    确认：0 崩溃、日志无 STUCK/Connection error、嵌入 embedding=X.XXXs（CPU 应 <0.1s）
# 4. 验证通过后再开自动：plist 改 HINDSIGHT_API_ENABLE_AUTO_CONSOLIDATION=true 并重启
```

## Pitfalls 与教训

- **health 通过 ≠ 系统正常**：`/health` 只查数据库，不查 LLM。LLM 不可用时服务仍 healthy。必须单独验证 LLM 连接。
- **"卡住" ≠ "僵尸"**：卡住（Connection error 重试）可能恢复；僵尸（payload_null）内容已丢才可删。删除前必须区分，别把有 payload 的活跃任务当僵尸删。
- **curl/httpx 通但 Hindsight 进程内 Connection error**：进程状态/连接池问题（进程在堵塞期间启动）。堵塞解除后重启 Hindsight 即可，新进程会打印 `Connection verified`。
- **KeepAlive=true 是双刃剑**：崩溃自动重启，但也是崩溃循环的放大器。修根因而不是依赖它。
- **嵌入走 CPU 是 Apple Silicon 本地服务的稳妥默认**：MPS 不稳定，服务用本地小模型（bge-small、MiniLM）时 CPU 足够，别为"加速"引入 MPS。

## 相关 Skill

- `hindsight-deployment`：部署、配置、worker 并发调优、模型兼容性
- 本 skill 聚焦崩溃/性能排查方法论，两者互补。

## 参考文档

- Hindsight GitHub: https://github.com/vectorize-io/hindsight
- 崩溃报告解析命令见本 skill 第 1 步