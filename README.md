# Hilight Skills

inSai Hilight 官方 AI Agent Skills 集合。让 AI Agent 获得 Hilight 平台的各项能力。

## Skills 目录

| Skill | 说明 |
|-------|------|
| [hilight-tiktok](skills/hilight-tiktok/) | 通过 Hilight MCP Server 生成 TikTok 带货视频，覆盖从素材上传到视频交付的全流程 |

## 快速开始

### 前置条件

- Hilight 平台账号和 API Key（在 [用户中心](https://app.hi-light.ai/userCenter) 获取）
- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 或 [OpenClaw](https://openclaw.com)

### 安装方式

- **自己动手安装** — 阅读 [手动安装指南](docs/setup.md)，按步骤运行命令即可
- **让 Agent 帮你装** — 将 [Agent 自动安装指南](docs/install-guide.md) 发给你的 AI Agent，它会自动完成配置（中途会向你索取 API Key）

## 项目结构

```
hilight-skills/
├── .claude-plugin/
│   └── plugin.json              # 插件元数据
├── docs/
│   ├── setup.md                 # 手动安装指南（给人看）
│   ├── install-guide.md         # Agent 自动安装指南（给 Agent 看）
│   └── usage-examples.md        # 使用示例
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md             # Skill 定义（必需）
│       ├── scripts/             # 可执行脚本（可选）
│       ├── references/          # 参考文档（可选）
│       └── assets/              # 模板、资源文件（可选）
├── template/
│   └── SKILL.md                 # 新 skill 模板
├── LICENSE
├── NOTICE
└── README.md
```

## 贡献新 Skill

1. 复制 `template/SKILL.md` 到 `skills/<your-skill-name>/SKILL.md`
2. 编写 YAML frontmatter（`name` + `description`）和 skill 内容
3. 按需添加 `scripts/`、`references/`、`assets/` 子目录
4. 在本 README 的 Skills 目录表中添加一行
5. 提交 PR

## License

Licensed under the [Apache License, Version 2.0](LICENSE).

Copyright 2026 inSai-Hilight. See [NOTICE](NOTICE) for details.
