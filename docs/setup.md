# Hilight TikTok — 手动安装指南

面向开发者和用户的手动安装指南。按照以下步骤配置 MCP Server 并安装 Skill。

## 前置条件

- Hilight 平台账号，在 [用户中心](https://app.hi-light.ai/userCenter) 获取 API Key（以 `sk_` 开头）
- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 或 [OpenClaw](https://openclaw.com)

## Claude Code

### 1. 配置 MCP Server

```bash
claude mcp add --transport http --scope user hilight https://wiseapi.hi-light.ai/mcp \
  --header "Authorization: Bearer sk_your_api_key_here"
```

> `--scope user` 使配置对所有项目生效，改为 `--scope project` 仅限当前项目。

验证连接：

```bash
claude mcp list
# hilight 应出现在列表中
```

### 2. 安装 Skill

**方式一：Plugin Marketplace（推荐）**

在 Claude Code 会话中执行：

```
/plugin marketplace add inSai-Hilight/skills
/plugin install hilight-tiktok@hilight-skills
```

**方式二：手动安装**

如果 `/plugin` 命令不可用，使用以下命令：

```bash
git clone https://github.com/inSai-Hilight/skills.git /tmp/hilight-skills
mkdir -p ~/.claude/skills
cp -r /tmp/hilight-skills/skills/hilight-tiktok ~/.claude/skills/hilight-tiktok
rm -rf /tmp/hilight-skills
```

### 3. 验证

确认 skill 文件已就位：

```bash
head -5 ~/.claude/skills/hilight-tiktok/SKILL.md
# 应输出包含 name: hilight-tiktok 的内容
```

重启 Claude Code，然后告诉它「帮我生成一个 TK 视频」，如果它开始询问商品信息，说明 Skill 已生效。

## OpenClaw

### 1. 安装 MCPorter（如未安装）

```bash
npm install -g mcporter
mcporter --version
```

### 2. 配置 MCP Server

```bash
mcporter config add hilight \
  --url "https://wiseapi.hi-light.ai/mcp" \
  --scope home
```

然后手动编辑 `~/.mcporter/mcporter.json`，添加鉴权 header：

```json
{
  "mcpServers": {
    "hilight": {
      "url": "https://wiseapi.hi-light.ai/mcp",
      "headers": {
        "Authorization": "Bearer sk_your_api_key_here"
      }
    }
  }
}
```

重启 Gateway：

```bash
openclaw gateway restart
```

验证连接：

```bash
mcporter config list
# hilight 应显示为已连接，提供 7 个工具
```

### 3. 安装 Skill

**方式一：ClawHub（推荐）**

```bash
npm i -g clawhub
clawhub install hilight-tiktok
```

> skill 默认安装到当前目录的 `./skills/` 下。安装完成后需重启 OpenClaw 会话才能生效。

**方式二：手动安装**

如果 skill 尚未发布到 ClawHub，使用以下命令：

```bash
git clone https://github.com/inSai-Hilight/skills.git /tmp/hilight-skills
cp -r /tmp/hilight-skills/skills/hilight-tiktok ~/.openclaw/skills/hilight-tiktok
rm -rf /tmp/hilight-skills
```

### 4. 验证

确认 skill 已安装：

```bash
openclaw skills list
# 应包含 hilight-tiktok
```

重启 OpenClaw 会话，告诉它「帮我生成一个 TK 视频」，如果它开始询问商品信息，说明 Skill 已生效。

## 可用工具

安装完成后，以下 7 个 MCP 工具可用：

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
| 连接被拒绝 | 检查网络连通性，确认可以访问 https://wiseapi.hi-light.ai |
| 401 未授权 | API Key 无效或已过期，前往 [Hilight 平台](https://app.hi-light.ai/userCenter) 重新生成 |
| 工具未显示 | 重启 Claude Code / OpenClaw gateway 并检查 MCP 服务状态 |
| 请求超时 | 服务可能负载较高，稍后重试 |
