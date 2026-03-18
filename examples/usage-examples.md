# Usage Examples

## Example 1: Complete Video Generation (All Info Provided)

**User:**
> 帮我生成2个TK带货视频，商品是"超轻跑鞋 Pro"，商品简介是"新一代碳板跑鞋，轻至180g，回弹提升40%"。用这3张图片：
> - https://cdn.example.com/shoe1.jpg
> - https://cdn.example.com/shoe2.jpg
> - https://cdn.example.com/shoe3.jpg
> 时长15秒，竖屏，英语。

**Agent Flow:**

```
→ create_project
  ✓ project_id: 12345

→ update_project(project_id=12345, product_name="超轻跑鞋 Pro",
    product_intro="新一代碳板跑鞋，轻至180g，回弹提升40%",
    images=["https://cdn.example.com/shoe1.jpg", ...],
    duration=1, count=2, ratio="9:16", language="en")
  ✓ Auto-generated selling points and target audience

→ get_pricing(project_id=12345)
  ✓ Unit price: 188⭐, Count: 2, Total: 376⭐, Balance: 1000⭐, Sufficient: true

Agent: "Generation will cost 376 starlight points (188 × 2). Your balance is 1000⭐. Confirm?"

User: "确认"

→ submit_project(project_id=12345)
  ✓ Submitted

→ query_tiktok_status(project_id=12345) [polling every 1 min]
  📊 Video 1: generating 45%
  📊 Video 2: generating 30%
  ...
  📊 Video 1: ✅ completed — https://cdn.example.com/video1.mp4
  📊 Video 2: ✅ completed — https://cdn.example.com/video2.mp4

Agent: "Both videos generated successfully! Download with:
  curl -o video1.mp4 'https://cdn.example.com/video1.mp4'
  curl -o video2.mp4 'https://cdn.example.com/video2.mp4'"
```

## Example 2: Incomplete Info (Agent Asks for Missing Fields)

**User:**
> 帮我生成一个TK视频

**Agent:**
> 好的！要生成 TikTok 带货视频，我需要以下信息：
> 1. **商品名称** — 你的产品叫什么？
> 2. **商品简介** — 简单描述一下产品特点
> 3. **商品图片** — 至少提供 1 张产品图片（URL 或直接上传）
>
> 另外，以下参数使用默认值，你可以根据需要修改：
> - 时长: ~15秒 | 数量: 1个 | 比例: 9:16 竖屏 | 语言: 英语

## Example 3: Balance Insufficient

```
→ get_pricing(project_id=12345)
  ✓ Total: 752⭐, Balance: 200⭐, Sufficient: false

Agent: "余额不足。生成需要 752 星光点，当前余额 200 星光点，差额 552 星光点。
请前往充值：https://app.hi-light.ai/userCenter/star-center
充值完成后告诉我，我将继续生成流程。"
```

## Example 4: Partial Failure and Retry

```
→ query_tiktok_status(project_id=12345)
  📊 Status: partial_failed
  Video 1: ✅ completed — https://cdn.example.com/video1.mp4
  Video 2: ❌ failed

Agent: "1个视频生成成功，1个失败。失败视频的星光点将自动退还。
是否要重试失败的视频？（重试将再次扣费 188 星光点）"

User: "重试"

→ retry_project(project_id=12345)
  ✓ Retried 1 video

→ query_tiktok_status(project_id=12345) [resume polling]
  📊 Video 2: generating 60%
  ...
  📊 Video 2: ✅ completed
```

## Example 5: Upload Local Images

**User:**
> 用这张图片生成视频 [attached base64 image]

**Agent:**

```
→ upload_image(image_data="data:image/jpeg;base64,/9j/4AAQ...")
  ✓ URL: https://cdn.example.com/uploaded-image.jpg, Status: pass

Agent: "Image uploaded and passed review. Now let me create the project..."
```

## Example 6: Modify Parameters After Pricing

```
→ get_pricing(project_id=12345)
  ✓ Total: 752⭐ (4 videos × 188⭐)

User: "太贵了，改成2个视频"

→ update_project(project_id=12345, count=2)
→ get_pricing(project_id=12345)
  ✓ Total: 376⭐ (2 videos × 188⭐)

Agent: "Updated to 2 videos, total cost 376⭐. Confirm?"
```
