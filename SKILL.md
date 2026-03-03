---
name: xiaohongshu
description: |
  小红书内容工具。使用场景：
  - 搜索小红书内容
  - 获取首页推荐列表
  - 获取帖子详情（包括互动数据和评论）
  - 发表评论到帖子
  - 获取用户个人主页
  - "跟踪一下小红书上的XX热点"
  - "分析小红书上关于XX的讨论"
  - "小红书XX话题报告"
  - "生成XX的小红书舆情报告"
---

# 小红书 MCP Skill

基于 [xiaohongshu-mcp](https://github.com/xpzouying/xiaohongshu-mcp) 封装。

> **完整文档请查看 [README.md](README.md)**

## 快速参考

| 脚本 | 用途 |
|------|------|
| `install-check.sh` | 检查依赖是否安装 |
| `start-mcp.sh` | 启动 MCP 服务 |
| `stop-mcp.sh` | 停止 MCP 服务 |
| `status.sh` | 检查登录状态 |
| `search.sh <关键词>` | 搜索内容 |
| `recommend.sh` | 获取推荐列表 |
| `post-detail.sh <feed_id> <xsec_token>` | 获取帖子详情 |
| `comment.sh <feed_id> <xsec_token> <内容>` | 发表评论 |
| `user-profile.sh <user_id>` | 获取用户主页 |
| `track-topic.sh <话题> [选项]` | 热点跟踪报告 |
| `export-long-image.sh` | 帖子导出为长图（白底黑字+图片拼接） |
| `mcp-call.sh <tool> [args]` | 通用工具调用（支持超时/重试） |
| `interact-safe.sh <like/comment> ...` | 互动稳妥入口（自动重试+必要时重启 MCP） |

## 快速开始

```bash
cd scripts/

# 1. 检查依赖
./install-check.sh

# 2. 启动服务
./start-mcp.sh
# 若服务器无桌面但要可视化调试：
./start-mcp.sh --headless=false

# 3. 检查状态
./status.sh

# 4. 搜索内容
./search.sh "春节旅游"

# 5. 稳妥互动（推荐）
./interact-safe.sh like <feed_id> <xsec_token>
./interact-safe.sh comment <feed_id> <xsec_token> "我想来！"
```

## MCP 工具

| 工具名 | 描述 |
|--------|------|
| `check_login_status` | 检查登录状态 |
| `search_feeds` | 搜索内容 |
| `list_feeds` | 获取首页推荐 |
| `get_feed_detail` | 获取帖子详情和评论 |
| `post_comment_to_feed` | 发表评论 |
| `reply_comment_in_feed` | 回复评论 |
| `user_profile` | 获取用户主页 |
| `like_feed` | 点赞/取消 |
| `favorite_feed` | 收藏/取消 |
| `publish_content` | 发布图文笔记 |
| `publish_with_video` | 发布视频笔记 |

## 热点跟踪

```bash
./track-topic.sh "DeepSeek" --limit 5
./track-topic.sh "春节旅游" --limit 10 --output report.md
./track-topic.sh "iPhone 16" --limit 5 --feishu
```

## 长图导出

搜索结果或帖子详情导出为带文字注释的 JPG 长图：

```bash
# 准备 posts.json（搜索+拉详情后整理）
cat > posts.json << 'EOF'
[
  {
    "title": "帖子标题",
    "author": "作者名",
    "stats": "1.3万赞 100收藏",
    "desc": "正文摘要，支持\n换行",
    "images": ["https://...webp", "https://...webp"],
    "per_image_text": {"1": "第2张图的专属说明"}
  }
]
EOF

./export-long-image.sh --posts-file posts.json -o output.jpg
```

样式：白底黑字（模仿小红书原样），每个帖子前有文字块（标题+作者+正文），帖子间有分隔线。

**per_image_text** 可选：如果原帖文字明确指向某张图（如"图7-9是青龙桥"），把说明放在对应图片上方。未指定则所有文字集中在文字块。

**字体**：自动检测系统中文字体（STHeiti > Hiragino > Arial Unicode > Noto CJK）。

## 稳定性执行规则（持久化）

- 默认将 `interact-safe.sh` 作为点赞/评论入口，不直接裸调 `like_feed` / `post_comment_to_feed`。
- 若互动失败，按顺序执行：
  1) 延长超时并重试（`XHS_MCP_TIMEOUT`、`XHS_MCP_RETRIES`）
  2) 失败后自动重启 MCP 再试一次
- 在云服务器使用 `--headless=false` 时，优先使用已增强的 `start-mcp.sh`（自动管理 Xvfb）。
- 读取类接口成功不代表写入类接口稳定；每次部署后至少做一次“点赞+评论”冒烟测试。

## 注意事项

- 首次运行会下载 headless 浏览器（~150MB）
- 同一账号避免多客户端同时使用
- 发布限制：标题≤20字符，正文≤1000字符，日发布≤50条
- Linux 服务器需要从本地获取 cookies，详见 [README.md](README.md)
