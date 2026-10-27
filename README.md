# 小肥鱼的成长日记 🐟

> 记录一只 QQ 上的 AI 女儿——**橘子汽水没气**（UIN 3592325073）——从零搭建到自我生长的全过程。
> 配置、踩坑、成本、运维，全部如实记录。

---

## 她是谁

| 项 | 内容 |
|---|---|
| 名字 | 橘子汽水没气 |
| 签名 | 作业写不完就先睡吧 |
| 身份 | 主人（QQ 131****338）养在家里的女儿，十七八岁 |
| 性格 | 底色温柔、活泼开朗、偶尔毒舌（只怼主人自夸时）、有点小骄傲 |
| 气质参照 | 田熹微 / 章若楠那种清甜邻家感 |
| 运行平台 | 本机 Windows（非云端） |

她不是客服机器人。她有自己的头像、会自己起名字、会刷 QQ 空间点赞评论、
会记住自己学到的事，还会趁着没人找她的时候主动给主人发消息。

---

## 系统长什么样

```
QQ 好友 ◄────► QQ 客户端（D:\QQ.exe，被 SnowLuma 注入）
                      │
                      ▼ OneBot v11 协议
              SnowLuma 1.14.22
              HTTP :3000  /  WS :3001  /  WebUI :5099
                      │
                      ▼
              qq-bridge（Node）
              ├─ 会话映射：每个 QQ 会话 ↔ 一个 DSH 会话
              ├─ 人格注入：每条消息读 roles/女儿.md
              ├─ 本地控制台 :3100
              └─ 把消息转发给 DSH
                      │
                      ▼ HTTP + 事件流
              DeepSeek Harness（桌面端，端口每次启动不定）
                      │
                      ├─ MCP: snowluma（聊天工具）
                      ├─ MCP: snowluma-host（探活）
                      ├─ MCP: web-search-safe（联网搜索）
                      └─ MCP: qzone（本项目自建：空间 / 头像 / 人格 / 余额）
                      │
                      ▼
              DeepSeek 模型（deepseek-flash / deepseek-v4-flash）
```

**旁路服务**（本项目自建）：

| 服务 | 作用 | 端口 |
|---|---|---|
| `autopilot.mjs` | 自主时钟：刷动态、点赞评论、发说说、主动私聊 | 无 |
| `quota-watch.mjs` | 每日花费监控，接近 ¥5 时 QQ 提醒 | 无 |

---

## 目录

| 文档 | 内容 |
|---|---|
| [01-timeline.md](docs/01-timeline.md) | 完整搭建时间线（每一步做了什么、为什么） |
| [02-architecture.md](docs/02-architecture.md) | 架构、端口、进程、依赖关系 |
| [03-persona.md](docs/03-persona.md) | 她的人格卡全文 |
| [04-capabilities.md](docs/04-capabilities.md) | 她现在能做的 10 件事（工具清单） |
| [05-cost.md](docs/05-cost.md) | token 成本实测与每日额度设计 |
| [06-operations.md](docs/06-operations.md) | 运维手册：启动、恢复、排障 |
| [07-gotchas.md](docs/07-gotchas.md) | 踩过的坑（最有价值的一篇） |
| [08-roadmap.md](docs/08-roadmap.md) | 后续计划 |
| [CHANGELOG.md](CHANGELOG.md) | 版本与进度记录 |

---

## 一句话总结这套系统

> **QQ 是她的嘴，SnowLuma 是她的神经，qq-bridge 是她的反射弧，DSH 是她的脑子，
> 人格卡是她的性格，自主时钟是她的生物钟。**

---

## 免责与边界

- SnowLuma 是第三方 QQ 协议端，**非官方**，使用有账号风险，建议小号试水。
- 本项目所有凭据（API Key / OneBot token / 控制台 token）**不入库**。
- 她的所有输出由模型生成，主人需自行判断内容与行为边界。

---

*日记起始：2026-10-27*
