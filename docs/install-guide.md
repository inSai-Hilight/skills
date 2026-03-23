# Hilight TikTok — Agent 自动安装指南

> **域名配置：**
>
> - `MCP_ENDPOINT` = `https://wiseapi.hi-light.ai/mcp`
> - `APP_URL` = `https://app.hi-light.ai`
> - `SKILL_REPO` = `https://github.com/inSai-Hilight/skills.git`
>
> 以下文档中所有 `{{MCP_ENDPOINT}}`、`{{APP_URL}}`、`{{SKILL_REPO}}` 均引用上方的值。

本文档供 AI Agent 阅读并自动完成 Hilight MCP Server 的安装和配置。

## 安装流程

### 第一步：获取 API Key

在继续之前，向用户索取 **Hilight API Key**（以 `sk_` 开头的 64 位十六进制字符串，在 {{APP_URL}}/userCenter 获取）。

> **必须等用户提供 API Key 后才能继续配置，不要使用占位符完成配置。**

### 第二步：检测当前 Agent 环境

根据当前运行环境选择对应的安装方式：

- **Claude Code** — 系统提示中包含 "You are Claude Code" → 按「Claude Code 安装」执行
- **OpenClaw** — 系统提示中包含 "OpenClaw" 或可调用 `openclaw` CLI → 按「OpenClaw 安装」执行

---

## Claude Code 安装

### 1. 配置 MCP Server

执行以下命令，将 `<API_KEY>` 替换为用户提供的实际值：

```bash
claude mcp add --transport http --scope user hilight {{MCP_ENDPOINT}} \
  --header "Authorization: Bearer <API_KEY>"
```

验证配置是否生效：

```bash
claude mcp list
```

确认输出中包含 `hilight`。

### 2. 安装 Skill

**优先使用 Plugin Marketplace 安装：**

```
/plugin marketplace add inSai-Hilight/skills
/plugin install hilight-tiktok@hilight-skills
```

**如果 `/plugin` 命令不可用或执行失败，必须使用手动方式安装（不可跳过）：**

```bash
git clone {{SKILL_REPO}} /tmp/hilight-skills
mkdir -p ~/.claude/skills
cp -r /tmp/hilight-skills/skills/hilight-tiktok ~/.claude/skills/hilight-tiktok
rm -rf /tmp/hilight-skills
```

### 3. 验证 Skill 安装

> **必须执行此步骤，不能跳过。Skill 未安装成功则整体安装未完成。**

检查 skill 文件是否存在：

```bash
cat ~/.claude/skills/hilight-tiktok/SKILL.md | head -5
```

如果输出包含 `name: hilight-tiktok`，则 Skill 安装成功。如果文件不存在，回到第 2 步重新安装。

### 4. 验证 MCP 连通性

提示用户重启 Claude Code（或执行 `/mcp` 重新加载），然后调用 `get_pricing` 测试连通性。返回错误响应（如"项目不存在"）即说明服务可达且鉴权正常。

---

## OpenClaw 安装

### 1. 检查并安装 MCPorter

先检查 MCPorter 是否已安装：

```bash
mcporter --version
```

如果命令不存在，执行安装：

```bash
npm install -g mcporter
```

确保配置目录存在：

```bash
mkdir -p ~/.mcporter
```

如果已安装，查找当前配置文件路径：

```bash
mcporter config list
```

### 2. 配置 MCP Server

将 `<API_KEY>` 替换为用户提供的实际值，编辑 `~/.mcporter/mcporter.json`（文件不存在则创建）：

```json
{
  "mcpServers": {
    "hilight": {
      "url": "{{MCP_ENDPOINT}}",
      "headers": {
        "Authorization": "Bearer <API_KEY>"
      }
    }
  }
}
```

验证连接：

```bash
mcporter config list
```

确认 `hilight` 显示为已连接状态，提供 7 个可用工具。

### 3. 安装 Skill

> **必须执行此步骤，不能跳过。Skill 未安装成功则整体安装未完成。**

**优先使用 GitHub 仓库安装：**

在 OpenClaw 会话中发送仓库链接，让 Agent 自动完成安装：

```
请帮我从 {{SKILL_REPO}} 安装 hilight-tiktok skill
```

**如果上述方式失败，手动安装（不可跳过）：**

```bash
git clone {{SKILL_REPO}} /tmp/hilight-skills
cp -r /tmp/hilight-skills/skills/hilight-tiktok ~/.openclaw/skills/hilight-tiktok
rm -rf /tmp/hilight-skills
```

### 4. 验证 Skill 安装

> **必须执行此步骤，不能跳过。Skill 未安装成功则整体安装未完成。**

```bash
openclaw skills list
```

确认输出中包含 `hilight-tiktok`。如果未显示，回到第 3 步重新安装。

### 5. 验证 MCP 连通性

```bash
mcporter call hilight get_pricing
```

返回错误响应（如"项目不存在"）即说明服务可达且鉴权正常。

---

## 安装完成后可用的工具

| 工具 | 说明 |
|------|------|
| `upload_image` | 上传 base64 图片至 CDN，自动内容审核 |
| `create_project` | 创建空的 TikTok 视频项目 |
| `update_project` | 设置/更新项目参数（合并语义） |
| `get_pricing` | 查询生成费用和余额 |
| `submit_project` | 提交生成（不可逆扣费） |
| `query_tiktok_status` | 查询生成进度 |
| `retry_project` | 重试所有失败的视频 |

## 常见问题

| 问题 | 解决方案 |
|------|----------|
| 连接被拒绝 | 检查网络连通性，确认可以访问 {{MCP_ENDPOINT}} |
| 401 未授权 | API Key 无效或已过期，引导用户前往 {{APP_URL}}/userCenter 重新生成 |
| 工具未显示 | 重启 Claude Code / OpenClaw gateway 并检查 MCP 服务状态 |
| 请求超时 | 服务可能负载较高，稍后重试 |

---

## ✅ 安装完成检查清单

Agent 收尾前根据当前环境逐项核对，全部通过才算安装完成。

### Claude Code

- [ ] `claude mcp list` 输出中包含 `hilight` 条目
- [ ] 调用 `get_pricing` 工具返回非网络错误（服务可达且鉴权正常）
- [ ] Skill 文件存在：`~/.claude/skills/hilight-tiktok/SKILL.md`

### OpenClaw

- [ ] `mcporter config list` 输出中包含 `hilight` 且显示已连接
- [ ] `mcporter call hilight get_pricing` 返回非网络错误（服务可达且鉴权正常）
- [ ] `openclaw skills list` 输出中包含 `hilight-tiktok`
