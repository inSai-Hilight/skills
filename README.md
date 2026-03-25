# Hilight Skills

inSai Hilight 官方 AI Agent Skills 集合。让 AI Agent 获得 Hilight 平台的各项能力。

## Skills 目录

| Skill | 说明 |
|-------|------|
| [hilight-tiktok](plugins/hilight-tiktok/skills/hilight-tiktok/) | 通过 Hilight MCP Server 生成 TikTok 带货视频，覆盖从素材上传到视频交付的全流程 |

## 快速开始

### 前置条件

- Hilight 平台账号和 API Key（在 [用户中心](https://app.hi-light.ai/userCenter) 获取）
- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 或 [OpenClaw](https://openclaw.com)

### Claude Code 安装

**1. 配置 MCP Server**

```bash
claude mcp add --transport http --scope user hilight https://wiseapi.hi-light.ai/mcp \
  --header "Authorization: Bearer sk_your_api_key_here"
```

**2. 安装 Skill（二选一）**

通过 Plugin Marketplace：

```
/plugin marketplace add inSai-Hilight/skills
/plugin install hilight-tiktok@hilight-skills
```

或手动安装：

```bash
git clone https://github.com/inSai-Hilight/skills.git /tmp/hilight-skills
mkdir -p ~/.claude/skills
cp -r /tmp/hilight-skills/plugins/hilight-tiktok/skills/hilight-tiktok ~/.claude/skills/hilight-tiktok
rm -rf /tmp/hilight-skills
```

**3. 重启 Claude Code，告诉它「帮我生成一个 TK 视频」**

### OpenClaw 安装

**1. 配置 MCP Server**

编辑 `~/.mcporter/mcporter.json`：

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

然后重启 Gateway：`openclaw gateway restart`

**2. 安装 Skill**

目前只支持手动安装

```bash
git clone https://github.com/inSai-Hilight/skills.git /tmp/hilight-skills
cp -r /tmp/hilight-skills/plugins/hilight-tiktok/skills/hilight-tiktok ~/.openclaw/skills/hilight-tiktok
rm -rf /tmp/hilight-skills
```

**3. 重启 OpenClaw 会话，告诉它「帮我生成一个 TK 视频」**

### 让 Agent 帮你装

将 [Agent 自动安装指南](docs/install-guide.md) 的链接发给你的 AI Agent，它会自动完成全部配置（中途会向你索取 API Key）。

> 详细安装说明请参考 [手动安装指南](docs/setup.md)

## 贡献新 Skill

1. 复制 `template/SKILL.md` 到 `skills/<your-skill-name>/SKILL.md`
2. 编写 YAML frontmatter（`name` + `description`）和 skill 内容
3. 按需添加 `scripts/`、`references/`、`assets/` 子目录
4. 在本 README 的 Skills 目录表中添加一行
5. 提交 PR

## License

Licensed under the [Apache License, Version 2.0](LICENSE).

Copyright 2026 inSai-Hilight. See [NOTICE](NOTICE) for details.
