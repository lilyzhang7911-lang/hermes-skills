---
name: engineering/performance-optimization
description: "性能优化工作流：先测量后优化，识别真 bottleneck，再验证回归。用于性能 SLA、Core Web Vitals、数据库查询、缓存、前后端调优。"
license: MIT
compatibility: hermes-agent
---

# 性能优化

## 铁律

**没有测量的优化是猜测。** 先建立 baseline，再定位真实瓶颈，再修复，再测量对比。只优化被测量证明重要的部分。

## 何时触发

- 有性能指标要求（响应时间 SLA、加载时间预算）
- 用户或监控报告慢行为
- Core Web Vitals 低于阈值
- 怀疑某次变更引入性能回退
- 处理大数据集或高流量场景

**何时不用**：没有证据先不要优化。过早优化会引入复杂度，代价超过收益。

## 工作流

```
1. MEASURE  → 用真实数据建立 baseline
2. IDENTIFY → 找到实际瓶颈（不是假设的）
3. FIX      → 修复具体瓶颈
4. VERIFY   → 再测量；保留或回滚
5. GUARD    → 加监控或测试防止回退
```

### Step 1 — 测量

两种互补方法，**都用**：

- **Synthetic**：Lighthouse / DevTools Performance，可控条件，可复现，适合 CI 回归
- **RUM**：真实用户数据，必须用来验证修复真的改善了体验

**后端最小可用测量**：

```bash
# 简单计时
console.time('db-query');
const result = await db.query(...);
console.timeEnd('db-query');
```

**前端最小可用测量**：

```bash
# Chrome DevTools → Performance tab → Record
# 或 Lighthouse CI
```

### Step 2 — 识别瓶颈

按症状定位：

| 症状 | 优先查 |
|------|--------|
| 首屏加载慢 | Bundle 体积、TTFB、渲染阻塞资源 |
| 交互卡顿 | 主线程长任务、重渲染 |
| 某个接口慢 | 数据库查询、N+1、缺失索引 |
| 内存持续增长 | 泄漏引用、无界缓存、大 payload |
| CPU 尖峰 | 同步重计算、正则回溯、外部依赖 |

### Step 3 — 修复常见反模式

只修**被测量证明的瓶颈**，不改其他地方。

#### N+1 查询

```python
# BAD: 循环内逐条查关联
tasks = db.query("SELECT * FROM tasks")
for t in tasks:
    t.owner = db.query("SELECT * FROM users WHERE id = ?", t.owner_id)

# GOOD: 一次 JOIN / include
tasks = db.query("""
    SELECT tasks.*, users.name as owner_name
    FROM tasks JOIN users ON tasks.owner_id = users.id
""")
```

#### 无界数据获取

```python
# BAD: 全表加载
all_tasks = db.query("SELECT * FROM tasks")

# GOOD: 分页 + 排序
tasks = db.query("""
    SELECT * FROM tasks
    ORDER BY created_at DESC
    LIMIT 20 OFFSET ?
""", ((page - 1) * 20,))
```

#### 缺失缓存

```python
# 对“读多写少”的数据加 TTL 缓存
CACHE_TTL = 5 * 60  # 5 分钟

async def get_app_config() -> AppConfig:
    now = time.time()
    if cached_config and now < cache_expiry:
        return cached_config
    cached_config = await db.fetch_one("SELECT * FROM config LIMIT 1")
    cache_expiry = now + CACHE_TTL
    return cached_config
```

#### 大响应体

```python
# BAD: 返回全字段，含敏感/大字段
return json.dumps(user.__dict__)

# GOOD: 只返回需要的字段
return json.dumps({
    "id": user.id,
    "name": user.name,
    "role": user.role,
})
```

### Step 4 — 验证（保留或回滚）

**一次只改一件事。** 三个优化一起提交只能得到一个数字，无法归因。

| 与 baseline 对比 | 动作 |
|-----------------|------|
| 超过阈值，测试绿 | **保留**。commit 里写清前后数值。 |
| 在噪声范围内 | **回滚**。中性改动也要还清复杂度债。 |
| 更差 | **回滚**。 |
| 更好了，但测试红了 | **回滚**。正确性优先于指标。 |

**中性就是回滚。** 你保留的代码要一直维护下去，它必须为自己的存在付账。

### Step 5 — 防止回退

- 把关键路径的**响应时间 p95** 加入 CI 或监控告警
- 对数据库慢查询加 `EXPLAIN` 审查（>100ms 的查询要在 code review 中被追问）
- 保留性能实验记录（PERF.md 或 PR description），避免同一个死idea 下季度被重提

## Hermes Agent 执行规范

### 测量工具选择

```python
# 后端 API 响应时间
terminal("curl -o /dev/null -s -w '%{time_total}\n' http://localhost:8000/api/endpoint")

# 数据库慢查询
terminal("psql -c 'SELECT * FROM pg_stat_activity WHERE state = \'active\''")

# 进程级资源
terminal("ps aux --sort=-%cpu | head -10")
terminal("ps aux --sort=-%mem | head -10")
```

### 分析代码路径

```python
# 用 search_files 定位候选瓶颈文件
search_files(pattern="SELECT .* FROM .* WHERE", target="content", path=".", file_glob="*.py")

# 用 read_file 检查关键函数
read_file(path="services/user_service.py", offset=1, limit=200)
```

### 验证修复

```python
# 跑基准测试或性能测试
terminal("pytest tests/performance/ -v")

# 对比前后响应时间
terminal("ab -n 100 -c 10 http://localhost:8000/api/endpoint")
```

## 常见陷阱

- ** premature optimization**：没有 profile 就改热点
- **缓存一切**：缓存是有状态的，缓存错误比慢更危险
- **忽略边界条件**：大数据量、空集合、并发下的表现往往不同
- **一次优化太多**：无法归因，无法回滚

## 完成定义

- [ ] 有 baseline 测量数据
- [ ] 瓶颈被明确识别（有证据）
- [ ] 修复只针对该瓶颈
- [ ] 修复后测量改善超过噪声
- [ ] 测试全绿
- [ ] 有监控或 CI 防止回退
