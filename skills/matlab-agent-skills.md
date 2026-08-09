---
name: matlab-agent-skills
description: MATLAB/Simulink 工程自动化技能中台。当用户需要 MATLAB/Simulink 任务的路由、执行、验证时加载此 skill。覆盖 9 个 specialist skills：orchestrator(路由)、runner(批处理)、data-analysis(数据分析)、simulink-modeling(Simulink 建模)、codegen-deploy(代码生成部署)、control-optimization(控制优化)、robotics-autonomy(机器人自主)、signal-vision-ai(信号视觉AI)、testing-ci(CI验证)。
category: devops
---

# MATLAB Agent Skills

## 触发条件
当用户请求涉及以下场景时加载此 skill：
- MATLAB/Simulink 任务路由与执行
- 代码生成 (MEX/CUDA/HDL)
- 控制系统设计/参数优化
- Simulink 建模仿真
- 信号处理/图像处理/深度学习实验
- 机器人/ROS/自主系统工作流
- MATLAB CI 验证与测试

## 核心架构

### Orchestrator → Specialist Router 模式
```
用户自然语言任务
    ↓
matlab-orchestrator (路由决策)
    ↓ 选择合适 skill
matlab-runner / codegen / control / signal / robotics / testing
    ↓
MCP 控制 MATLAB/Simulink 执行
    ↓
产物落盘 + 验收检查
```

### 9 个 Skill 能力矩阵

| Skill | 职责定位 | Hermes 对应模式 |
|---|---|---|
| `matlab-orchestrator` | 任务入口与路由中枢，MCP auto mode 优先 | ↔ `multi-mcp-agent-router` / `advisor-orchestrator-worker` |
| `matlab-runner` | 批处理执行、日志采集、产物落盘 | ↔ `terminal(background=true)` + `process(action="poll")` |
| `matlab-data-analysis` | 数据处理、统计分析、拟合建模、图表导出 | ↔ `execute_code` (Python) + `browser_vision` |
| `matlab-simulink-modeling` | Simulink 模型创建/修改/仿真，可视化优先 | ↔ `mcp_chrome_devtools_mac_*` (可见窗口交互) |
| `matlab-codegen-deploy` | MEX/CUDA/HDL 代码生成与部署验证 | ↔ `terminal` + `patch` (自编译验证) |
| `matlab-control-optimization` | PID/MPC/鲁棒控制、参数辨识、约束优化 | ↔ `execute_code` (数值计算) |
| `matlab-robotics-autonomy` | ROS/导航/UAV/传感器融合工作流 | ↔ `delegate_task` (并行机器人仿真) |
| `matlab-signal-vision-ai` | 信号处理/图像视觉/深度学习实验 | ↔ `browser_get_images` + `vision_analyze` |
| `matlab-testing-ci` | 单元测试、CI 验证与报告输出 | ↔ `test-driven-development` (RED-GREEN-REFACTOR) |

## 执行规则

### MCP 优先原则
1. **默认使用官方 MATLAB MCP auto mode**，复用已有共享会话
2. 仅当用户明确要求 headless/batch 时才使用 `matlab -batch`
3. 如果 MCP 不可用，报告 blocker 而非静默切换执行模式

### Self-Compile and Verify
生成代码后必须自动编译验证：
1. MATLAB Coder: `codegen` → MEX 输出 vs MATLAB golden output 对比
2. Simulink Coder: `slbuild` → 检查生成的 `.c/.h/project/elf/axf/hex/bin` 非空
3. 嵌入式目标: 先尝试配置的硬件 toolchain，失败则捕获精确缺失信息并生成 portable C fallback

### 验证闭环
任务完成标准：
1. 脚本/函数无错误运行
2. Simulink 模型加载/更新/编译无 diagram errors
3. 仿真到达指定 stop time
4. 关键 logged signals 非空且有限
5. 生成的 figures/data/reports 存在且非空
6. 代码生成产物检查存在且大小 > 0
7. 任何步骤失败时记录精确 blocker、失败命令、最高价值下一步行动

## 本地路径
- 仓库位置: `/Users/sunwenning/Desktop/MyWorkHome/matlab-agent-skills/matlab-agent-skills-main`
- Obsidian 归档: `20_Projects/matlab-agent-skills-analysis.md`

## Pitfalls
- PowerShell (.ps1) 脚本在 macOS 不原生支持，需 WSL/Cross-platform 适配
- 无 MATLAB R2026a 授权环境时部分能力不可用
- GitHub Actions workflow + MathWorks 在线文档在翻墙受限环境下可能无法访问
- Simulink 可视化优先：默认打开可见窗口让用户观看过程，而非仅返回最终文件
