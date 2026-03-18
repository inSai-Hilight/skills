# Hilight MCP Server — TikTok Video Generation

通过 MCP (Model Context Protocol) 协议让 AI Agent 直接调用 [inSai Hilight](https://app.hi-light.ai) 平台的 TikTok 带货视频生成能力。

## 功能概述

提供 7 个 MCP Tool，覆盖 TikTok 视频生成的完整链路：

1. **upload_image** — 上传商品图片（base64 → CDN），自动内容审核
2. **create_project** — 创建视频项目
3. **update_project** — 设置项目参数（商品名、简介、图片、时长、数量等），支持 merge 更新
4. **get_pricing** — 查询生成费用和星光点余额
5. **submit_project** — 提交生成（不可逆扣费）
6. **query_tiktok_status** — 查询生成进度
7. **retry_project** — 重试失败的视频

## 典型流程

```
用户: "帮我用这3张图生成一个TK视频，商品是超轻跑鞋Pro"

Agent:
  1. upload_image × 3  → 获取 CDN URL
  2. create_project     → 获取 project_id
  3. update_project     → 设置参数 + 自动生成卖点/受众
  4. get_pricing        → 展示费用，等待确认
  5. submit_project     → 提交生成
  6. query_tiktok_status (每分钟轮询) → 展示进度
  7. 完成 → 展示视频下载链接
```

## 前置条件

- Hilight 平台账号和 API Key（在 [用户中心](https://app.hi-light.ai/userCenter) 获取）
- 足够的星光点余额

## 安装

### Claude Code

**方式一：CLI 命令（推荐）**

```bash
claude mcp add-json hilight '{"type":"http","url":"https://<SERVER_HOST>:10621/mcp","headers":{"Authorization":"Bearer sk_your_api_key_here"}}'
```

添加 `--scope user` 使配置对所有项目生效，或 `--scope project` 仅限当前项目。

**方式二：手动编辑配置文件**

用户级配置（所有项目生效）— 编辑 `~/.claude.json`：

```json
{
  "mcpServers": {
    "hilight": {
      "type": "http",
      "url": "https://<SERVER_HOST>:10621/mcp",
      "headers": {
        "Authorization": "Bearer sk_your_api_key_here"
      }
    }
  }
}
```

项目级配置（仅当前项目生效）— 在项目根目录创建 `.mcp.json`：

```json
{
  "mcpServers": {
    "hilight": {
      "type": "http",
      "url": "https://<SERVER_HOST>:10621/mcp",
      "headers": {
        "Authorization": "Bearer ${HILIGHT_API_KEY}"
      }
    }
  }
}
```

> **提示：** `.mcp.json` 支持环境变量展开（`${VAR}` 或 `${VAR:-default}`），推荐将 API Key 存入环境变量后在配置中引用，避免明文提交到版本控制。

验证配置：

```bash
claude mcp list
```

**安装 Skill**

将 `skills/hilight-tiktok/` 目录复制到 `~/.claude/skills/`，然后重启 Claude Code。

### OpenClaw

```bash
mcporter config add hilight \
  --type http \
  --url "https://<SERVER_HOST>:10621/mcp" \
  --header "Authorization: Bearer sk_your_api_key_here"

openclaw gateway restart
```

详细安装指南请参考 [AGENT_INSTALL.md](./AGENT_INSTALL.md)。

## 参数说明

### update_project 参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| project_id | number | 是 | 项目 ID |
| product_name | string | 否 | 商品名称 |
| product_intro | string | 否 | 商品简介 |
| images | string[] | 否 | 商品图片 URL 数组 |
| duration | number | 否 | 时长：1=~15s, 2=~30s, 3=~1min, 4=~25s |
| count | number | 否 | 生成数量（最多 4） |
| ratio | string | 否 | 画面比例："9:16", "16:9", "1:1" |
| language | string | 否 | 语言代码："en", "vi", "th" 等 |
| sell_points | string | 否 | 卖点文本 |
| target_audience | string | 否 | 目标受众文本 |
| video_requirements | string | 否 | 额外创作要求 |
| remove_fields | string[] | 否 | 要清空的字段名 |

## 计费说明

- 每个视频消耗固定的星光点
- 提交项目时一次性扣费，扣费不可逆
- 生成失败的视频会自动退还星光点
- 重试失败视频会再次扣费
- 通过 API 生成的记录在星光点流水中显示为"通过API调用消耗"

## 项目结构

```
hilight-mcp/
├── README.md              # 本文件
├── AGENT_INSTALL.md       # Agent 自动安装指南
├── skills/
│   └── hilight-tiktok/
│       └── SKILL.md        # Skill 文件（意图识别 + 流程编排）
└── examples/
    └── usage-examples.md  # 使用示例
```

## License

Proprietary — inSai Hilight
