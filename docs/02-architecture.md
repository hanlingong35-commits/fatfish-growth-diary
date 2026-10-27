# 架构与端口

## 进程清单

| 进程 | 作用 | 启动方式 |
|---|---|---|
| `QQ.exe`（多个子进程） | QQ 客户端，SnowLuma 的注入目标 | 开机自启（唯一保留的应用自启） |
| `SnowLuma/node.exe index.mjs` | QQ 协议端（OneBot v11） | 由 `fix-dsh-port.ps1` 拉起 |
| `qq-bridge/node.exe src/bridge.js` | 消息桥接 + 人格注入 + 本地控制台 | 同上 |
| `qq-bridge/node.exe src/autopilot.mjs` | 自主时钟 | 同上 |
| `qq-bridge/node.exe src/quota-watch.mjs` | 额度监控 | 同上 |
| `DeepSeek Harness.exe` | DSH 桌面端（agent 大脑） | 由 `start-all.ps1` 确保运行 |
| MCP servers（4 个 node 进程） | DSH 拉起的工具服务 | DSH 自动 |

## 端口

| 端口 | 服务 | 说明 |
|---|---|---|
| 3000 | OneBot HTTP API | SnowLuma |
| 3001 | OneBot WebSocket | SnowLuma |
| 5099 | SnowLuma WebUI | 管理面板 |
| 3100 | qq-bridge 控制台 | 需 token（`state/console-token`） |
| **每次不同** | DSH Web | 56288 → 64303 → 58945 → 55826，每次启动随机 |

> ⚠️ DSH 端口每次重启都变，这是设计如此。桥接配置写死端口，所以需要 `fix-dsh-port.ps1` 自动对齐。

## 启动顺序（强依赖）

```
QQ.exe  ──必须先有──►  SnowLuma（注入 QQ 进程）
                              │
                              ▼ 3000/3001
                        qq-bridge  ──需要──►  DSH（任一端口）
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
              autopilot            quota-watch
              （需 SnowLuma）       （只读本地账本）
```

**顺序不能乱**：QQ 没起 → SnowLuma 注入失败 → OneBot 端口不开 → 桥接连不上。

## 数据流

**收消息**：
```
好友发 QQ → QQ 客户端 → SnowLuma(Hook) → OneBot WS → qq-bridge
  → 命中白名单？→ 映射到 DSH 会话 → DSH agent（注入人格卡 + 工具）
  → agent 回复 → qq-bridge → OneBot HTTP → QQ
```

**自主行为**：
```
autopilot 每 90 分钟醒来
  → 拉 qzone feeds → 调模型决定点赞/评论（或跳过）
  → 偶尔让模型生成说说正文 → qzone_post
  → 偶尔生成一句话 → OneBot 私聊主人
```

## 关键文件

| 路径 | 作用 |
|---|---|
| `qq-bridge/config.json` | 主配置（白名单、模型、额度、autopilot） |
| `qq-bridge/roles/女儿.md` | 人格卡（每条消息注入，热生效） |
| `qq-bridge/state/current-role.json` | 当前启用的人格 |
| `qq-bridge/state/token-usage.jsonl` | token 账本（quota-watch 和 account_balance 都读它） |
| `qq-bridge/state/sessions.json` | QQ 会话 ↔ DSH 会话映射 |
| `SnowLuma/config/onebot_<UIN>.json` | OneBot 端口与 token |
| `SnowLuma/config/runtime.json` | `hookAutoLoad` 必须为 true |
| `~/.dsh/profiles/desktop/cordis.patch.yml` | MCP server 与 preset 注册 |

## 自建代码

| 文件 | 行数 | 说明 |
|---|---|---|
| `src/mcp-qzone-safe.js` | ~600 | 自建 MCP server：空间 + 头像 + 人格 + 余额（10 个工具） |
| `src/autopilot.mjs` | ~330 | 自主时钟 |
| `src/quota-watch.mjs` | ~130 | 额度监控 |
| `scripts/gen-avatar.py` | ~230 | 按描述画头像（Pillow） |
| `fix-dsh-port.ps1` | ~150 | 端口对齐 + 全组件恢复 |
| `start-all.ps1` | ~90 | 开机启动链 |
