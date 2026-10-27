# 她现在能做什么（10 个工具）

工具来自两个 MCP server：`snowluma`（qq-bridge 自带）和 `qzone`（本项目自建）。

## 聊天类（snowluma 自带）

| 工具 | 作用 |
|---|---|
| `qq_send_message` | 发一条或多条消息（数组=分多条），可引用、可 @ |
| `qq_reply` | 引用回复某条消息 |
| `qq_send_burst` | 分多条发送，间隔像真人 |
| `qq_get_unread_messages` | 看未读 |
| `qq_get_recent_messages` | 看最近消息（可翻页） |
| `qq_wait_for_messages` | 等待/沉睡前观察（5 分钟长轮询） |
| `qq_mark_read` | 标记已读 |
| `qq_set_wake_config` | 设置唤醒条件（@ / 名字 / 关键词 / 某人发言 / 拍一拍） |
| `qq_send_poke` | 拍一拍 |
| `qq_memory_*` | 轻量记忆：活跃话题 / 想说的话 / 对某人的印象 |
| `qq_slang_*` | 黑话查询与提交 |
| `qq_report_feedback` | 向控制台反馈问题 |

## 表情包

| 工具 | 作用 |
|---|---|
| `qq_list_stickers` | 看自己的收藏表情库 |
| `qq_get_sticker_image` | 看某张表情长什么样（视觉模型能看懂） |
| `qq_send_sticker` | 发一张收藏表情 |
| `qq_collect_sticker` | **收藏当前会话里别人发的表情**（她只在觉得有趣时才收） |
| `qq_sticker_note` | 给她收藏的表情写自己的理解 |
| `qq_get_message_images` | 看消息里的图片/表情 |

**当前收藏库**：2 张种子（她自己的形象图 + 橘猫头像），由 `add_custom_face` 导入。
以后她在群里遇到好玩的会自己收，慢慢长大。

## QQ 空间（本项目自建）

| 工具 | 作用 | 限流 |
|---|---|---|
| `qzone_get_feeds` | 看好友动态 | 只读 |
| `qzone_list_my_posts` | 看自己的说说 | 只读 |
| `qzone_like` | 点赞 | 10/时 · 40/天 |
| `qzone_comment` | 评论 | 5/时 · 20/天 |
| `qzone_post` | 发说说 | 3/天 |

**安全护栏**：只能操作「拉取过的 tid」，凭空编一个会被拒绝。

## 自我管理（本项目自建）

| 工具 | 作用 | 限制 |
|---|---|---|
| `qq_set_nickname` | 改自己的昵称 + 个性签名 | 1–36 字 |
| `qq_set_avatar` | 换自己的头像 | 本地路径或 URL |
| `avatar_generate` | **按心情画一张新头像**（猫/鲸鱼/星星/云/心/月亮 + 主色） | 生成后要再调 `qq_set_avatar` |
| `persona_evolve` | 往自己的「成长笔记」写一条 | ≤100 字/条，≤800 字总量 |
| `account_balance` | 查余额 + 今日花费 + 还能用几天 | 只读 |

---

## 她做不了的事（有意的边界）

- ❌ 删除说说、拉黑、改可见范围
- ❌ 管理群（禁言、踢人、改群设置）
- ❌ 发文件、下载文件
- ❌ 改别人的资料
- ❌ 越出白名单找陌生人说话（`allow.private` / `allow.groups` 之外一律拒绝）

## 控制台

`http://127.0.0.1:3100`（token 在 `state/console-token`）

- 运行总览 / 会话与审批 / 人格与角色 / 令牌与花费 / 访问与安全 / 调试运维
- 可以切换模式、改白名单、看每个群每个好友的 token 消耗
