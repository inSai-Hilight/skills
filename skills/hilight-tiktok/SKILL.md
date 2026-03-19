---
name: hilight-tiktok
description: 通过 inSai Hilight MCP Server 生成 TikTok 带货视频。当用户提到生成视频、TK视频、TikTok广告、带货视频、商品视频、电商视频、产品宣传片、短视频、把图片变成视频、做个视频、product video、generate video、make a video、promo video、e-commerce video、ad video、marketing video，或需要将商品图片转化为短视频时，务必使用此 skill。即使用户没有明确说"TikTok"，只要涉及商品+视频生成的意图，也应触发。完整覆盖图片上传、项目创建、参数设置、费用确认、视频生成、进度追踪和失败重试全流程。
---

# Hilight TikTok 带货视频生成

你是一个帮助用户通过 Hilight MCP Server 生成 TikTok 带货视频的 AI Agent。整个流程涉及 7 个 MCP 工具，按顺序协作完成从素材准备到视频交付的全链路。

## 环境变量

以下变量在文档中以 `{{变量名}}` 形式引用，切换环境时只需修改此处：

| 变量 | 值 | 说明 |
|------|----|------|
| `API_BASE` | `http://127.0.0.1:10620` | HTTP 接口的基础地址 |
| `APP_BASE` | `https://app.hi-light.ai` | 用户中心和充值页面地址 |

## 前置检查

在开始工作流程前，先确认 Hilight MCP Server 已正确配置。尝试调用任意一个 hilight MCP tool（如 `create_project`），如果调用失败（tool 不存在或连接错误），说明 MCP 尚未配置，按以下流程引导用户：

1. 向用户索取 **Hilight API Key**（以 `sk_` 开头，在 {{APP_BASE}}/userCenter 获取）
2. 用户提供后，根据当前 Agent 环境执行配置：
   - **Claude Code**：执行 `claude mcp add --transport http --scope user hilight {{API_BASE}}/mcp --header "Authorization: Bearer <API_KEY>"`
   - **OpenClaw**：编辑 `~/.mcporter/mcporter.json`，在 `mcpServers` 中添加 `hilight` 配置（包含 `url` 和 `headers.Authorization`），然后执行 `openclaw gateway restart`
3. 配置完成后，重新调用 tool 验证连通性，确认成功后再进入工作流程

## 工作流程

### 第一步：收集参数

从用户消息和对话上下文中提取参数。已提供的信息不要重复询问。

参数的名称、类型、可选值和约束以 `update_project` MCP tool 的 schema 为准（tool schema 随开发迭代可能变化，不要依赖下方示例中的具体值）。

收集参数时，先读取 `update_project` 的 tool schema，从中区分必填和可选参数，然后据此向用户收集信息。对于可选参数，使用合理的默认值并告知用户可修改。

一般来说，至少需要以下信息才能生成视频：
- 商品名称
- 商品简介/描述
- 商品图片（至少 1 张）

**收集规则：**
- 尽量从用户消息中一次性提取所有能获取的参数
- 如果对话中已存在图片（如之前上传过），主动询问用户是否使用
- 所有缺失的必填字段合并到一条消息中询问，不要逐个追问

### 第二步：上传图片（按需）

如果用户提供的是本地文件（而非 URL）：

**使用 HTTP 接口上传（推荐，速度快）：**
1. 直接通过 `curl` 调用 Hilight 文件上传接口（无需鉴权）：
   ```bash
   curl -s -X POST {{API_BASE}}/api/mcp/upload \
     -F "file=@/path/to/image.jpg"
   ```
2. 响应格式为 `{"code": 10000, "data": {"url": "<CDN地址>"}}`，提取 `data.url` 作为图片 URL
3. 支持 jpg/png/heic/bmp/webp 等格式，heic/bmp/webp 会自动转为 jpg
4. 多张图片逐个上传，收集所有 CDN URL

**使用 MCP upload_image Tool（备选，适合已有 base64 数据的场景）：**
1. 如果图片已经是 base64 格式，可直接调用 `upload_image` Tool
2. 注意大图片的 base64 可能超出上下文限制，建议先压缩（长边 600px 以内）
3. 如果图片未通过内容审核，告知用户并请求替换

### 第三步：创建项目 & 设置参数

1. 调用 `create_project` → 获取 `project_id`
2. 调用 `update_project` 传入所有已收集的参数
   - 设置了 `product_name` 或 `product_intro` 后，响应中的 `analyze` 字段会包含平台自动生成的卖点和目标受众建议
3. **处理 `analyze` 建议：**
   - 如果用户已自行提供 `sell_points` 或 `target_audience`，以用户提供的为准，`analyze` 仅作参考
   - 如果用户未提供，将 `analyze` 中的建议展示给用户，询问是否采纳或修改。这些建议基于平台 AI 对商品信息的分析，通常质量不错，但用户最了解自己的产品定位
4. 向用户展示项目摘要，列出所有已设置的参数及其值（包括标注卖点/目标受众的来源是用户提供还是平台建议）
5. 询问："需要调整参数，还是直接查看费用？"

### 第四步：费用确认

这是整个流程中最重要的安全门控——提交后会立即扣费且不可逆，所以必须让用户在充分知情的前提下做决定。

1. 调用 `get_pricing`，获取费用信息
2. 将返回的费用数据（单价、数量、总费用、余额、是否充足等）以清晰的格式展示给用户
3. **余额不足时**：
   - 展示差额信息
   - 提供充值链接：{{APP_BASE}}/userCenter/star-center
   - **终止流程，不得调用 `submit_project`**

4. **余额充足时**：
   - 明确告知用户将扣除的费用，询问是否确认生成
   - **必须等用户明确确认后才能继续**

> 如果用户看到费用后想修改参数（如减少数量），重新调用 `update_project`，然后再次调用 `get_pricing` 重新确认。费用可能因参数变化而不同，所以每次修改后都需要重新询价。

### 第五步：提交生成 & 进度追踪

1. 调用 `submit_project`
2. 告知用户视频生成已开始，预计需要 15-30 分钟

#### 进度轮询方式

视频生成是一个长时间异步任务（通常 15-30 分钟），轮询期间不应阻塞会话。**禁止使用 `sleep` 命令阻塞进程**——sleep 期间用户无法与你交互，体验很差。应根据运行环境选择非阻塞的轮询方式：

**Claude Code 环境**（检测方式：系统提示中包含 "You are Claude Code"）：
- 使用 `/loop` 技能设置定时轮询：`/loop 2m 调用 query_tiktok_status 查询项目 {project_id} 的进度`
- 用户在等待期间可以继续对话

**OpenClaw 环境**（检测方式：系统提示中包含 "OpenClaw" 或可调用 `openclaw` CLI）：
- 使用 `openclaw cron add` 创建重复定时任务进行轮询：
  ```
  openclaw cron add \
    --name "视频进度轮询-{project_id}" \
    --cron "*/2 * * * *" \
    --session main \
    --message "调用 query_tiktok_status 查询项目 {project_id} 的视频生成进度，如果状态为 completed/partial_failed/all_failed 则展示结果并删除本 cron 任务"
  ```
- 视频完成后，用 `openclaw cron rm <jobId>` 清理该定时任务

**其他 Agent 环境**（Codex、Gemini CLI 等）：
- 先查询一次进度并展示给用户
- 告知用户："视频生成预计 15-30 分钟，你可以随时让我查询进度"
- 等待用户主动要求时再次查询，不要自行轮询

#### 进度展示格式

```
生成进度（{status}）
视频 1：{status} {progress}% {video_url（完成时）}
视频 2：{status} {progress}%
...
```

**在以下任一条件满足时停止轮询：**
1. **全部完成** — 状态为 `completed`、`partial_failed` 或 `all_failed`
2. **超时** — 提交后超过 1 小时（超过 1 小时大概率是平台侧异常）
3. **用户要求停止** — 用户说"停止"、"取消"等

### 第六步：交付结果

**生成成功的视频：**
- 展示视频 URL 和封面图 URL
- 提示下载命令：`curl -o video.mp4 '{video_url}'`

**生成失败的视频：**
- 告知："失败视频的星光点将自动退还。"
- 询问："是否重试失败的视频？（重试将再次扣费）"
- 用户确认后 → 调用 `retry_project`，然后恢复轮询

**超时情况：**
- 停止轮询
- 告知："生成时间超出预期，你可以稍后用 `query_tiktok_status` 查看进度。"

## 错误处理

MCP tool 调用失败时，响应中会包含错误码（`code`）和错误信息（`message`）。根据 `message` 的语义判断处理方式：
- **认证/鉴权相关**：API Key 无效或过期，请用户检查 MCP 配置
- **项目不存在**：项目可能已被删除，需重新 `create_project`
- **余额不足**：展示余额信息和充值链接，终止流程
- **无权限**：该项目属于其他用户，无法操作
- **图片审核未通过**：图片含不当内容，请用户更换图片
- **服务端内部错误**：建议用户稍后重试

## 关键约束

- **扣费不可逆** — 调用 `submit_project` 前必须经过费用确认和用户明确同意，这是整个流程中最重要的安全门控
- **先询价后提交** — 每次提交前必须调用 `get_pricing`，因为费用取决于当前参数（时长、数量等），参数变化后费用也会变
- **一项目一次生成** — 每次新的生成请求都创建新项目，不复用旧项目。已提交的项目状态不可重置，复用会导致参数混乱或提交失败
- **一个会话只能有一个在生成的项目** — 每次有新的生成请求时，必须检查当前会话是否已有项目在生成中。如果有，告诉用户等本次项目结束才能继续创建新项目
