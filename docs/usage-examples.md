# 使用示例

## 示例 1：信息完整，一次性生成

**用户：**
> 帮我生成2个TK带货视频，商品是"超轻跑鞋 Pro"，商品简介是"新一代碳板跑鞋，轻至180g，回弹提升40%"。用这3张图片：
> - https://cdn.example.com/shoe1.jpg
> - https://cdn.example.com/shoe2.jpg
> - https://cdn.example.com/shoe3.jpg
> 时长15秒，竖屏，英语。

**Agent 执行流程：**

```
→ create_project
  ✓ project_id: 12345

→ update_project(project_id=12345, product_name="超轻跑鞋 Pro",
    product_intro="新一代碳板跑鞋，轻至180g，回弹提升40%",
    images=["https://cdn.example.com/shoe1.jpg", ...],
    duration=1, count=2, ratio="9:16", language="en")
  ✓ 自动生成卖点和目标受众

→ get_pricing(project_id=12345)
  ✓ 单价: 188⭐, 数量: 2, 总费用: 376⭐, 余额: 1000⭐, 充足: true

Agent: "生成费用为 376 星光点（188 × 2），当前余额 1000⭐。确认生成？"

User: "确认"

→ submit_project(project_id=12345)
  ✓ 已提交

→ query_tiktok_status(project_id=12345) [每分钟轮询]
  📊 视频 1：生成中 45%
  📊 视频 2：生成中 30%
  ...
  📊 视频 1：✅ 完成 — https://cdn.example.com/video1.mp4
  📊 视频 2：✅ 完成 — https://cdn.example.com/video2.mp4

Agent: "2个视频全部生成成功！下载命令：
  curl -o video1.mp4 'https://cdn.example.com/video1.mp4'
  curl -o video2.mp4 'https://cdn.example.com/video2.mp4'"
```

## 示例 2：信息不完整，Agent 主动询问

**用户：**
> 帮我生成一个TK视频

**Agent：**
> 好的！要生成 TikTok 带货视频，我需要以下信息：
> 1. **商品名称** — 你的产品叫什么？
> 2. **商品简介** — 简单描述一下产品特点
> 3. **商品图片** — 至少提供 1 张产品图片（URL 或直接上传）
>
> 另外，以下参数使用默认值，你可以根据需要修改：
> - 时长: ~15秒 | 数量: 1个 | 比例: 9:16 竖屏 | 语言: 英语

## 示例 3：余额不足

```
→ get_pricing(project_id=12345)
  ✓ 总费用: 752⭐, 余额: 200⭐, 充足: false

Agent: "余额不足。生成需要 752 星光点，当前余额 200 星光点，差额 552 星光点。
请前往充值：https://app.hi-light.ai/userCenter/star-center
充值完成后告诉我，我将继续生成流程。"
```

## 示例 4：部分失败并重试

```
→ query_tiktok_status(project_id=12345)
  📊 状态: partial_failed
  视频 1：✅ 完成 — https://cdn.example.com/video1.mp4
  视频 2：❌ 失败

Agent: "1个视频生成成功，1个失败。失败视频的星光点将自动退还。
是否要重试失败的视频？（重试将再次扣费 188 星光点）"

User: "重试"

→ retry_project(project_id=12345)
  ✓ 重试 1 个视频

→ query_tiktok_status(project_id=12345) [恢复轮询]
  📊 视频 2：生成中 60%
  ...
  📊 视频 2：✅ 完成
```

## 示例 5：上传本地图片

**用户：**
> 用这张图片生成视频 [附带 base64 图片]

**Agent 执行流程：**

```
→ upload_image(image_data="data:image/jpeg;base64,/9j/4AAQ...")
  ✓ URL: https://cdn.example.com/uploaded-image.jpg, 审核状态: 通过

Agent: "图片上传成功，审核通过。接下来创建项目..."
```

## 示例 6：询价后修改参数

```
→ get_pricing(project_id=12345)
  ✓ 总费用: 752⭐ (4 个视频 × 188⭐)

User: "太贵了，改成2个视频"

→ update_project(project_id=12345, count=2)
→ get_pricing(project_id=12345)
  ✓ 总费用: 376⭐ (2 个视频 × 188⭐)

Agent: "已改为 2 个视频，总费用 376⭐。确认生成？"
```
