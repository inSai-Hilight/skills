# Hilight MCP Server — Agent Installation Guide

This guide is for AI agents to automatically install and configure the Hilight MCP Server.

## Prerequisites

- A valid Hilight API Key (starts with `sk_`, 64-character hex string)
- The Hilight MCP Server endpoint URL

## Claude Code Installation

### 1. Configure MCP Server

Add the following to `~/.claude/settings.json` under the `mcpServers` key:

```json
{
  "mcpServers": {
    "hilight": {
      "type": "streamableHttp",
      "url": "https://<SERVER_HOST>:10621/mcp",
      "headers": {
        "Authorization": "Bearer <API_KEY>"
      }
    }
  }
}
```

Replace:
- `<SERVER_HOST>` — The Hilight server hostname (ask the user or check their environment)
- `<API_KEY>` — The user's Hilight API Key

### 2. Install Skill File

Copy the skill file to the Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills
cp skills/hilight-tiktok.md ~/.claude/skills/hilight-tiktok.md
```

Or write the skill file content directly if cloning is not possible.

### 3. Verify Installation

After restarting Claude Code (or running `/mcp` to reload):

1. Check MCP Server is connected: the `hilight` server should appear in the MCP servers list
2. Call `get_pricing` with any project_id to test connectivity — an error response (e.g., "项目不存在") confirms the server is reachable and authentication works

## OpenClaw Installation

### 1. Configure MCP Server via MCPorter

```bash
mcporter config add hilight \
  --type streamableHttp \
  --url "https://<SERVER_HOST>:10621/mcp" \
  --header "Authorization: Bearer <API_KEY>"
```

Or manually edit `~/.config/mcporter/mcporter.json`:

```json
{
  "servers": {
    "hilight": {
      "type": "streamableHttp",
      "url": "https://<SERVER_HOST>:10621/mcp",
      "headers": {
        "Authorization": "Bearer <API_KEY>"
      }
    }
  }
}
```

### 2. Restart Gateway

```bash
openclaw gateway restart
```

### 3. Verify

```bash
mcporter list
```

The `hilight` server should appear as connected with 7 tools available.

### 4. Install Skill (Optional)

OpenClaw skills can be placed in the project's skills directory or loaded via configuration. Refer to OpenClaw documentation for skill installation.

## Available Tools

After installation, the following 7 tools are available:

| Tool | Description |
|------|-------------|
| `upload_image` | Upload base64 image to CDN with content review |
| `create_project` | Create an empty TikTok video project |
| `update_project` | Set/update project parameters (merge semantics) |
| `get_pricing` | Calculate cost and check balance |
| `submit_project` | Submit for generation (irreversible, deducts points) |
| `query_tiktok_status` | Query generation progress |
| `retry_project` | Retry all failed videos |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection refused | Check if the MCP Server is running on port 10621 |
| 401 Unauthorized | API Key is invalid or expired. Regenerate at Hilight platform. |
| Tools not showing | Restart Claude Code / OpenClaw gateway and check MCP server status |
| Timeout errors | The server may be under load. Retry after a moment. |
