# 运维手册

## 一键启动

| 场景 | 操作 |
|---|---|
| 开机自启 | 全自动（启动文件夹 `启动QQ女儿.vbs` → `start-all.ps1`） |
| DSH 单独重启后 | 双击 `修复DSH端口.bat` |
| 手动全量恢复 | `powershell -File qq-bridge\start-all.ps1` |

## 启动链

```
start-all.ps1
 ├─ 确保 DSH 在跑（没跑就启动，轮询等端口）
 └─ fix-dsh-port.ps1
     ├─ [0] QQ 没跑就启动（SnowLuma 的注入目标）
     ├─ [0] 强制 SnowLuma hookAutoLoad = true
     ├─ [1] SnowLuma（没跑就拉起，等 3001）
     ├─ [2] 扫端口认出 DSH → 改 config.json 的 baseUrl
     ├─ [3] 桥接（端口变了才重启）
     ├─ [4] 自主时钟
     └─ [5] 额度监视
```

## 停止

| 目标 | 方法 |
|---|---|
| 只停桥接 | 结束 `src/bridge.js` 的 node 进程 |
| 全部停 | 结束所有 `node.exe`（会连带停掉 SnowLuma/autopilot/quota） |
| 让她下线（不动 DSH） | 退出 SnowLuma 即可 |

## 日志

| 日志 | 路径 |
|---|---|
| 启动链 | `qq-bridge\state\start-all.log` |
| 桥接 | `qq-bridge\bridge.out.log` |
| 自主时钟 | `qq-bridge\state\autopilot.log` |
| 额度 | `qq-bridge\state\quota-watch.log` |
| SnowLuma | `SnowLuma\_launch*.out.log` + `SnowLuma\logs\snowluma-*.log` |
| QQ 空间活动 | `qq-bridge\state\qq-activity.log` |
| 工具调用 | `qq-bridge\state\tool-calls.jsonl` |

## 常见故障

### QQ 发消息没反应

按顺序查：

1. **桥接在跑吗？** → `Get-Process node` / 看 `bridge.out.log`
2. **SnowLuma 通吗？** → 3000/3001 端口；看日志有没有 `[Hook] login detected`
3. **QQ 在跑吗？** → SnowLuma 没有注入目标就起不来
4. **DSH 端口对得上吗？** → `config.json` 的 `dsh.baseUrl` vs 实际端口
5. **白名单？** → `allow.private` / `allow.groups` 是空的话**一条都不响应**
6. **DSH 侧通吗？** → `node src/self-test.js http://127.0.0.1:<端口>`

### 她「失联」了（进程都在但收不到消息）

SnowLuma 的 `[Hook]` 会退化：进程和 WebSocket 都活着，但收不到 QQ 数据。
看 SnowLuma 日志里有没有：

```
WARN [Hook] receive path stale: ... silentFor=136193ms; reporting good=false
```

处理：重启 SnowLuma（`fix-dsh-port.ps1` 会做）。

### SnowLuma 起不来

- 检查 `config/runtime.json` 的 `hookAutoLoad` 是否为 `true`
- 检查 QQ 是否在运行
- 检查 3001 端口是否被占

### 桌面端换了端口，桥接连不上

双击 `修复DSH端口.bat`。它会自动扫端口（靠 401 响应体里那句
`dsh web authentication required` 认人）。

### 昵称/头像改了但看起来没变

用 `get_stranger_info`（带 `no_cache`）查服务端：
```
POST /get_stranger_info  {"user_id":3592325073,"no_cache":true}
```
如果服务端是新名字，那就是**QQ 客户端缓存**或**好友备注**在显示旧值。

### 额度提醒没收到

- `config.json` 的 `ownerQQ` 是否已设
- SnowLuma 是否在线（提醒走 OneBot 私聊）
- 看 `state\quota-watch.log` 有没有「已提醒主人」

## 备份什么

| 必须备份 | 原因 |
|---|---|
| `qq-bridge\config.json` | 全部配置 |
| `qq-bridge\roles\` | 人格卡（她的灵魂） |
| `qq-bridge\state\` | 会话映射、成长笔记、记账、收藏表情索引 |
| `SnowLuma\config\` | OneBot 端口与 token |
| `~\.dsh\profiles\desktop\` | DSH 的 MCP 与 preset 注册 |

> `state\startup-backup\*.reg` 是自启项备份，误删可还原。
