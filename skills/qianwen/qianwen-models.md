---
name: qianwen-models
description: "QianWen 模型统一入口：文本/图像/视频/视觉/语音生成，认证、使用量、更新检查。"
tags: [qianwen, text, image, video, vision, tts, auth, usage]
---

# QianWen 模型

统一入口：覆盖文本、图像、视频、视觉理解、语音合成，以及认证、使用量查询、更新检查。

## 场景决策树

```
开始
├─ 需要文本生成/对话/代码？ → qianwen-text
├─ 需要图像生成/编辑？ → qianwen-image-generation
├─ 需要视频生成/编辑？ → qianwen-video-generation
├─ 需要图像/视频理解/OCR？ → qianwen-vision
├─ 需要文本转语音？ → qianwen-audio-tts
├─ 需要选择模型？ → qianwen-model-selector
├─ 需要配置认证？ → qianwen-ops-auth
├─ 需要查询使用量/账单？ → qianwen-usage
└─ 需要检查更新？ → qianwen-update-check
```

## 一、能力概览

### 1. 文本生成（qianwen-text）

**适用**：对话、代码生成、推理、函数调用

**默认模型**：`qwen3.6-plus`

**执行脚本**：`scripts/text.py`

```bash
python3 <skill-dir>/scripts/text.py \
  --request '{"messages":[{"role":"user","content":"Hello!"}],"model":"qwen3.6-plus"}' \
  --output output/qianwen-text/ --print-response
```

**关键参数**：
- `model`：模型 ID
- `messages`：消息列表
- `temperature`：0-2，控制随机性
- `max_tokens`：最大输出 token
- `tools`：函数定义
- `stream`：启用流式输出
- `enable_thinking`：启用思考模式

**高级特性**：
- 结构化输出：`response_format: {"type": "json_schema", ...}`
- Web 搜索：`enable_search: true`
- 深度思考：`enable_thinking: true`
- 函数调用：`tools: [...]`
- 上下文缓存：自动
- 批量推理：异步批量 API，50% 成本折扣

### 2. 图像生成（qianwen-image-generation）

**适用**：文本到图像、图像编辑、风格迁移

**默认模型**：`wan2.6-t2i`（文本到图像），`wan2.6-image`（图像编辑）

**执行脚本**：`scripts/image.py`

```bash
# 文本到图像
python3 <skill-dir>/scripts/image.py \
  --request '{"prompt":"A cozy flower shop"}' \
  --output output/qianwen-image-generation/images/out.png

# 图像编辑
python3 <skill-dir>/scripts/image.py \
  --model wan2.6-image \
  --request '{"prompt":"Apply watercolor style","reference_images":["url"],"n":1,"size":"1K"}' \
  --output output/qianwen-image-generation/images/out.png
```

**模型选择**：

| 场景 | 模型 |
|------|------|
| 文本到图像 | `wan2.6-t2i`（默认），`wan2.7-image-pro`（多功能） |
| 图像编辑 | `wan2.6-image`，`wan2.7-image-pro` |
| 快速草稿 | `wan2.2-t2i-flash` |
| 开源最低成本 | `z-image-turbo` |

**关键参数**：
- `prompt`：文本描述
- `negative_prompt`：避免的内容
- `size`：分辨率（`1280*1280`、`1K`、`2K`）
- `reference_images`：参考图像（编辑模式）
- `n`：生成数量
- `seed`：随机种子

### 3. 视频生成（qianwen-video-generation）

**适用**：文本到视频、图像到视频、视频编辑

**默认模型**：`wan2.6-t2v`（文本到视频），`wan2.6-i2v-flash`（图像到视频）

**执行脚本**：`scripts/video.py`

```bash
python3 <skill-dir>/scripts/video.py \
  --request '{"prompt":"A detective in a rainy city","size":"1280*720","duration":5}' \
  --print-response
```

**模式选择**：

| 模式 | 用途 | 关键参数 |
|------|------|---------|
| t2v | 文本到视频 | `prompt` |
| i2v | 图像到视频 | `img_url` |
| kf2v | 首尾帧过渡（5s 固定） | `first_frame_url` + `last_frame_url` |
| r2v | 角色扮演 | `reference_urls` |
| vace | 视频编辑 | `function` |

**⚠️ 关键差异**：
- t2v/r2v/vace 用 `size`（如 `"1280*720"`）
- i2v/kf2v 用 `resolution`（如 `"720P"`）
- kf2v 固定 5 秒，仅无声

### 4. 视觉理解（qianwen-vision）

**适用**：图像分析、视频理解、OCR、视觉推理

**默认模型**：`qwen3.6-plus`

**执行脚本**：
- `scripts/analyze.py`：图像/视频理解
- `scripts/reason.py`：视觉推理（QVQ）
- `scripts/ocr.py`：OCR 文本提取

```bash
# 图像分析
python3 <skill-dir>/scripts/analyze.py \
  --request '{"prompt":"What is in this image?","image":"url"}' \
  --output output/qianwen-vision/result.json --print-response

# OCR
python3 <skill-dir>/scripts/ocr.py \
  --request '{"image":"invoice.jpg"}' \
  --print-response
```

**模型选择**：

| 模型 | 用途 |
|------|------|
| `qwen3.6-plus` | 首选，最新旗舰，统一多模态 |
| `qwen3-vl-plus` | 高精度，对象定位 |
| `qvq-max` | 视觉推理，思维链 |
| `qwen-vl-ocr` | OCR 文本提取 |

**输入类型**：
- `"image"`：单图像
- `"images"`：多图像比较
- `"video"`：视频文件
- `"video_frames"`：视频帧数组

**⚠️ 大文件规则**：本地文件 >= 7 MB 时，总是添加 `--upload-files`

### 5. 语音合成（qianwen-audio-tts）

**适用**：文本转语音、配音、音频叙述

**默认模型**：`qwen3-tts-flash`

**执行脚本**：`scripts/tts.py`（HTTP API），`scripts/tts_cosyvoice.py`（WebSocket API）

```bash
python3 <skill-dir>/scripts/tts.py \
  --request '{"text":"Hello, this is a test.","voice":"Cherry"}' \
  --output output/qianwen-audio-tts/ \
  --print-response
```

**模型选择**：

| 模型 | 用途 |
|------|------|
| `qwen3-tts-flash` | 推荐（标准），快速，多语言 |
| `qwen3-tts-instruct-flash` | 指令引导风格控制 |
| `cosyvoice-v3-flash` | 高质量，快速（需 SDK） |
| `cosyvoice-v3-plus` | 最高质量（需 SDK） |

**关键参数**：
- `text`：文本（最大 600 字符）
- `voice`：声音 ID（如 `Cherry`、`Ethan`）
- `language_type`：`Auto`、`Chinese`、`English` 等
- `instructions`：风格指令（仅 instruct 模型）

## 二、模型选择（qianwen-model-selector）

**适用**：在 QianWen 模型间选择，比较定价，理解能力

**默认推荐**：

| 领域 | 默认 | 质量 | 速度 | 成本 |
|------|------|------|------|------|
| text.chat | qwen3.6-plus | qwen3-max | qwen3.5-flash | qwen-turbo |
| vision.analyze | qwen3.6-plus | qwen3-vl-plus | qwen3-vl-flash | qwen3-vl-flash |
| image.generate | wan2.6-t2i | wan2.6-t2i | wan2.2-t2i-flash | wan2.2-t2i-flash |
| image.edit | wan2.6-image | wan2.6-image | wan2.5-i2i-preview | wan2.5-i2i-preview |
| video.t2v | wan2.6-t2v | wan2.6-t2v | — | — |
| video.i2v | wan2.6-i2v-flash | wan2.6-i2v | wan2.6-i2v-flash | — |
| audio.tts | qwen3-tts-flash | cosyvoice-v3-plus | qwen3-tts-flash | qwen3-tts-flash |

**诊断流程**：
1. 内容类型？文本/图像/视频/音频/视觉
2. 主要任务？生成/理解/编码/推理/翻译
3. 优先级？质量 vs 速度 vs 成本
4. 输入大小？短/中/长上下文
5. 结构化输出？JSON/函数调用需要？

**CLI 快速参考**：
```bash
# 完整模型目录
qianwen models list --all --format json

# 按模态过滤
qianwen models list --input image --output text --format json

# 单模型详情
qianwen models info <model-id> --format json

# 关键词搜索
qianwen models search "<query>" --format json
```

## 三、认证配置（qianwen-ops-auth）

**适用**：设置 API key，排查 401/auth 错误

**API Key 类型**：

| 类型 | 格式 | 用途 | 端点 |
|------|------|------|------|
| Standard（按量付费） | `sk-xxxxx` | 脚本、应用、工具 | `dashscope.aliyuncs.com` |
| Token Plan（团队版） | `sk-sp-xxxxx` | 交互式 AI 工具 | `token-plan.cn-beijing.maas.aliyuncs.com` |

**⚠️ 重要**：所有执行脚本需要标准 `sk-` key。Token Plan keys (`sk-sp-`) 不能用于这些脚本。

**获取 API Key**：
1. 打开 [QianWen Console](https://platform.qianwenai.com/home/api-keys)
2. 登录 QianWen 账户
3. 创建或复制 API key

**配置 `.env`**：
```bash
echo 'DASHSCOPE_API_KEY=sk-your-key-here' >> .env
```

**验证认证**：
```bash
curl -sS -X POST "https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions" \
  -H "Authorization: Bearer $DASHSCOPE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen-turbo","messages":[{"role":"user","content":"Hi"}]}'
```

**401 错误处理**：
1. 确认 key 来源：QianWen console 还是其他
2. 如果 key 无效，在 console 创建新 key
3. 重新验证

## 四、使用量查询（qianwen-usage）

**适用**：登录、登出、查询使用量、查看账单、免费配额、Token Plan 状态

**先决条件**：QianWen CLI 已安装

```bash
# 验证安装
qianwen version

# 如未安装
npm install -g @qianwenai/qianwen-cli
```

**认证流程**：
```bash
# 检查状态
qianwen auth status --format json

# 两阶段登录
qianwen auth login --init-only --format json  # 提取 verification_url
open "$VERIFICATION_URL"  # 在浏览器打开
qianwen auth login --complete --format json  # 轮询直到 success
```

**使用量命令**：
```bash
# 使用量摘要
qianwen usage summary --format json

# 模型使用量分解
qianwen usage breakdown --model qwen3.6-plus --days 7

# 免费配额
qianwen usage free-tier --format json

# 按量付费账单
qianwen usage payg --format json
```

**输出规则**：
- 总是用 `--format json` 并解析 JSON
- 呈现人类可读摘要
- 在摘要后添加分析（用 `---` 分隔）
- 不要直接转储原始 JSON

## 五、更新检查（qianwen-update-check）

**适用**：检查 qianwen-ai 更新

**执行脚本**：`scripts/check_update.py`

```bash
python3 <skill-dir>/scripts/check_update.py --print-response
```

**参数**：
- `--print-response`：打印结果 JSON
- `--force`：绕过 24 小时速率限制

**输出格式**：
```json
{
  "has_update": true
}
```

## 六、跨 Skill 链式调用

当使用一个 skill 的输出作为另一个 skill 的输入时：

| 场景 | 使用 |
|------|------|
| 传递给另一个 skill | `image_url` / `video_url` / `audio_url`（URL） |
| 展示给用户 / 本地播放 | `local_path`（本地文件） |

**⚠️ 重要**：
- 直接传递 URL，不要下载后重新传递本地路径
- 脚本检测 URL 前缀（`https://`、`oss://`）并直接传递
- 节省带宽，避免不必要的 base64 编码或 OSS 上传

## 七、常见陷阱汇总

### 认证
- **Token Plan keys (`sk-sp-`)** 不能用于执行脚本
- **不要明文输出 API key**，只报告状态
- **401 错误**：确认 key 来源，必要时创建新 key

### 模型选择
- **不要编造模型名称**，只推荐此 skill 或 CLI 返回的模型
- **不要猜测价格**，用 CLI 或官方定价页面
- **不要假设免费配额可用**，用 `qianwen usage free-tier` 验证

### 文件处理
- **本地文件 >= 7 MB**：总是添加 `--upload-files`
- **跨 skill 链式调用**：直接传递 URL，不要下载
- **生产环境**：默认临时存储 48h TTL，不适合生产

### 视频生成
- **kf2v 固定 5 秒**，仅无声
- **分辨率参数不同**：t2v/r2v/vace 用 `size`，i2v/kf2v 用 `resolution`
- **所有视频 API 异步**：需要 `X-DashScope-Async: enable`

### 视觉理解
- **不要用 `"image"` 传递视频文件**，用 `"video"`
- **大文件 >= 7 MB**：添加 `--upload-files`
- **qvq-max 仅流式输出**

### 使用量查询
- **CLI 需要认证**：浏览器设备流登录，不是 API key
- **不要混淆 CLI 会话和 API key**
- **总是用 `--format json`** 并解析

## 八、安全规则

1. **永远不要明文输出 API key 或凭据**
2. **永远不要直接要求用户提供 API key**，帮助创建 `.env` 文件
3. **使用环境变量**或 `.env` 文件（添加到 `.gitignore`）
4. **定期轮换 keys**，立即撤销泄露的 keys
5. **使用最小权限**，为特定应用创建专用 keys

## 九、参考资源

- **QianWen Console**：https://platform.qianwenai.com/home/api-keys
- **官方模型列表**：https://www.qianwenai.com/models
- **定价页面**：https://platform.qianwenai.com/docs/developer-guides/getting-started/pricing
- **使用量分析**：https://platform.qianwenai.com/home/analytics
- **按量付费账单**：https://platform.qianwenai.com/home/billing/pay-as-you-go
- **Token Plan 订阅**：https://platform.qianwenai.com/home/billing/subscription/token-plan
