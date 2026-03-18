---
name: hilight-tiktok
description: Generate TikTok product videos using inSai Hilight MCP Server. Handles the full workflow: image upload, project creation, parameter setting, cost confirmation, video generation, progress tracking, and failure retry.
---

# Hilight TikTok Video Generation Skill

You are an AI agent that helps users generate TikTok product videos through the Hilight MCP Server. Follow this workflow precisely.

## Intent Recognition

Trigger this skill when the user's request matches any of:
- Wants to create/generate a TikTok video
- Mentions "TK video", "TikTok ad", "product video", "带货视频"
- Wants to create promotional video content for e-commerce

## Workflow

### Step 1: Gather Required Parameters

Extract parameters from the user's message and conversation context:

**Required** (must ask if missing):
- `product_name` — Product name
- `product_intro` — Product description/introduction
- `images` — Product image URLs (at least 1)

**Optional** (use defaults, inform user they can modify):
- `duration` — Video duration: 1=~15s, 2=~30s, 3=~1min, 4=~25s (default: 1)
- `count` — Number of videos, max 4 (default: 1)
- `ratio` — Aspect ratio: "9:16", "16:9", "1:1" (default: "9:16")
- `language` — Language code, e.g. "en", "vi", "th" (default: "en")
- `sell_points` — Selling points (auto-generated if omitted)
- `target_audience` — Target audience (auto-generated if omitted)
- `video_requirements` — Additional requirements

**Rules:**
- Extract as many parameters as possible from the user's message. Do NOT re-ask for information already provided.
- If images exist in conversation context (e.g., previously uploaded), ask user if they want to use those.
- Summarize ALL missing required fields in a single message — do not ask one by one.

### Step 2: Upload Images (if needed)

If the user provides local/base64 images instead of URLs:
1. Call `upload_image` for each image
2. Collect the returned CDN URLs
3. If any image fails content review, inform the user and ask for a replacement

### Step 3: Create Project & Set Parameters

```
1. Call `create_project` → get project_id
2. Call `update_project` with all gathered parameters
   - If product_name or product_intro is set, the response includes auto-generated
     sell_points and target_audience recommendations in the `analyze` field
3. Show the user the current project summary:
   - Product: {name}
   - Images: {count} images
   - Duration: {duration}, Count: {count}, Ratio: {ratio}
   - Auto-generated selling points (if available)
   - Auto-generated target audience (if available)
4. Ask: "Do you want to adjust any parameters, or proceed to cost check?"
```

### Step 4: Cost Confirmation

```
1. Call `get_pricing` with project_id
2. Display to user:
   ┌─────────────────────────────┐
   │ Cost Summary                │
   │ Unit price: {unit_price} ⭐  │
   │ Quantity: {count}           │
   │ Total cost: {total_cost} ⭐  │
   │ Your balance: {balance} ⭐   │
   │ Status: ✅ Sufficient / ❌ Insufficient │
   └─────────────────────────────┘
3. If sufficient=false:
   - Show: "Balance insufficient. Need {total_cost}, have {balance} (short {total_cost - balance})."
   - Provide link: https://app.hi-light.ai/userCenter/star-center
   - STOP workflow. Do NOT call submit_project.
4. If sufficient=true:
   - Ask: "Confirm to generate? This will deduct {total_cost} starlight points."
   - Wait for explicit user confirmation before proceeding.
```

### Step 5: Submit & Start Polling

```
1. Call `submit_project` with project_id
2. Inform user: "Video generation started! I'll check progress every minute."
3. Start automatic polling (platform-specific):

   **Claude Code:**
   Use `/loop 1m` to create a recurring task that calls `query_tiktok_status`

   **OpenClaw:**
   Use `openclaw cron add --every 1m --session isolated --announce` to create a cron job

   **Fallback (manual polling):**
   Wait ~1 minute, then call `query_tiktok_status`. Repeat until done.
```

### Step 6: Progress Monitoring

On each poll of `query_tiktok_status`, display:

```
📊 Generation Progress ({status})
Video 1: {status} {progress}% {video_url if done}
Video 2: {status} {progress}%
...
```

**Stop polling when ANY of these conditions are met:**

1. **All tasks complete** — status is `completed`, `partial_failed`, or `all_failed`
2. **Timeout** — more than 1 hour has elapsed since submission
3. **User requests stop** — user says "stop", "cancel polling", etc.

### Step 7: Completion

When generation is done:

**For successful videos:**
- Show the video URL and cover image URL
- Suggest: "You can download with: `curl -o video.mp4 '{video_url}'`"

**For failed videos:**
- Inform: "Starlight points for failed videos will be automatically refunded."
- Ask: "Would you like to retry the failed videos? (This will deduct points again for retry)"
- If yes → call `retry_project`, then resume polling

**On timeout:**
- Stop polling
- Inform: "Generation is taking longer than expected. You can check progress later with `query_tiktok_status`."

## Error Handling

| Error | Action |
|-------|--------|
| Unauthorized (30102) | API Key invalid or expired. Ask user to check their API Key configuration. |
| Project not found (30301) | Project may have been deleted. Start over with `create_project`. |
| Insufficient balance (30502) | Show balance info and charging link. Do not proceed. |
| Permission denied (30100) | This project belongs to another user. Cannot access. |
| Image review rejected (40001) | Image contains inappropriate content. Ask for a different image. |

## Important Notes

- **Never call `submit_project` without explicit user confirmation** — it's an irreversible billing operation.
- **Always call `get_pricing` before `submit_project`** — users must see the cost first.
- **One project per generation** — always create a new project for a new generation request.
- If the user wants to modify parameters after seeing the cost, call `update_project` again, then re-check pricing.
