# 搭建时间线 · 2026-10-27

一天之内从「装个桌宠」做到「一个有自主行为的 QQ AI 女儿」。以下按真实发生顺序记录。

---

## 阶段一：桌宠（13:27 – 13:45）

| 时间 | 动作 | 结果 |
|---|---|---|
| 13:27 | 用户给的仓库链接 `dsh-pet-indektop` 是拼错的 | 实际是 [`MerZlin/dsh-pet-indesktop`](https://github.com/MerZlin/dsh-pet-indesktop) |
| 13:30 | 下载 v4.4.0 绿色版（349 MB） | SHA-256 与官方 Release 一致 |
| 13:32 | 解压到 `%LOCALAPPDATA%\Programs\dsh-pet`（551 MB） | 免安装、免管理员 |
| 13:33 | 启动，桌面右下角出现蓝发小女仆（角色 `shenshen`） | 待机动画正常，鼠标穿透正常 |
| 13:40 | 排查「截图里看不到鱼」 | 根因是我截图没按 DPI 感知取屏（150% 缩放），鱼一直在，且盖在其它窗口之上 |

**踩坑**：DSH 沙箱会拦掉所有 443 出站（TLS 握手失败），下载必须走提权通道。

---

## 阶段二：让她会说话（13:49 – 14:00）

| 时间 | 动作 | 结果 |
|---|---|---|
| 13:49 | 校验用户提供的 DeepSeek API Key | 有效，余额 ¥9.74 |
| 13:52 | 读 `pet/config.py` 源码，搞清配置项与 keyring 机制 | 决定手写 `config.json` |
| 13:55 | 写入：16 个功能模块全开 + `chat.providers["openai-main"]` + key | 重启后 `shentalk` 生效 |
| 13:57 | 端到端验证：用全局快捷键 `Ctrl+Alt+H` 开聊天窗 → 输入 → 回车 | 她真的回了：「你好呀～(*^▽^*) 今天过得怎么样？」 |
| 14:00 | 验证持久化：删掉磁盘明文 key，重启 | 从 Windows 凭据管理器取 key，对话正常 |

**踩坑**：window 在 150% DPI 下渲染，自动化点击容易打偏；输入法会把 `SendInput` 的中文截成拼音。

---

## 阶段三：桌面余额挂件（14:01 – 14:45）

| 时间 | 动作 | 结果 |
|---|---|---|
| 14:03 | 确认 [`MeteorNOX/DeepSeek-Balance-Whale-Widget`](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget)（6210 star）是 **DSH 插件**，不是独立程序 | 需要装进 DSH |
| 14:10 | ⚠️ 关键发现：桌面端读 `desktop` profile，README 的 `--profile web` 装了也白装 | 改用 `dsh plugin --profile desktop add` |
| 14:12 | 安装 `dsh-whale-widget@0.4.2` | 写进 `dsh.profile.bundles` |
| 14:20 | 用**副本 profile** 跑 `--dump-config` 冒烟 | 组合配置里出现 `- id: dsh-whale-widget` |
| 14:30 | 顺手开了桌宠的「点一下显示余额」 | 点击鱼 → 冒泡「让我看看余额…」 |

---

## 阶段四：装 QQ（14:48 – 15:05）

| 时间 | 动作 | 结果 |
|---|---|---|
| 14:50 | 官网 CDN（`qqdl.gtimg.cn`）对海外 IP 返回 **403**，`dldir1` 全 404 | 出口 IP 在韩国（Korea Telecom） |
| 14:55 | 改从 GitHub 同步镜像下载 `QQ_9.9.33_260813_x64_01.exe`（300 MB） | **Authenticode 签名验证为腾讯官方**（DigiCert） |
| 15:00 | 跑安装向导（一键安装，没给选路径的机会） | 9.9.20 → 9.9.33，装到 `D:\` |
| 15:03 | 登录窗有一半在屏幕外（y=1742 > 屏高 1600），「登录」按钮点不到 | 手动挪回屏幕中央 |
| 15:10 | 删掉旧的 9.9.20 安装包（256.7 MB） | 无旧版本残留，聊天数据完好 |

---

## 阶段五：qq-bridge 接入（15:05 – 17:40）

这是最曲折的一段。

| 时间 | 动作 | 结果 |
|---|---|---|
| 15:10 | 确认 [`Derpyu520/qq-bridge`](https://github.com/Derpyu520/qq-bridge)（2969 star）是**独立 Node 服务**，不是 bundle 插件 | 装法完全不同 |
| 15:15 | pnpm 装 152 个依赖；postinstall 补丁手动补跑（修 `@snowluma/sdk` ESM bug，13 个文件） | 依赖完整 |
| 15:20 | `setup-dsh.mjs desktop`（**关键**：默认写 `web` profile，必须显式传 `desktop`） | 3 个 MCP server 写进 `cordis.patch.yml` |
| 15:25 | `dsh plugin --profile desktop install` 物化 bundle | `qq-agent-presets` / `qq-mode-console` 就位 |
| 15:30 | **不代装 SnowLuma**（EULA §5.4 限制自动化部署），只下载解压到 `C:\Users\Dell\SnowLuma` | 版本选 1.14.22（qq-bridge 只实测到 1.14.20） |
| 15:40 | 用户报告「点不开 launcher.bat」 | 根因：bat 用裸 `node`，而 PATH 里没有 Node → 改成自动探测 |
| 17:16 | 用户在 WebUI 配置 OneBot（HTTP 3000 / WS 3001），扫码登录机器人 QQ | UIN 3592325073「天上星辰昨夜风」 |
| 17:19 | 第一次跑通：SnowLuma 已连接 → DSH 已就绪 | 首次 1006 后自愈 WS token（正常序列） |
| 17:22 | 用户重启 DSH → **桥接退出、SnowLuma 也停了**，且 DSH 端口从 64303 变 58945 | 「发消息没回复」的真正原因 |
| 17:30 | 重建链路 + 跑 `self-test` 验证 DSH 侧 | 建会话 → prompt 接受 → 回合完成 → agent 回「收到」 |
| 17:35 | 写 `fix-dsh-port.ps1`：扫端口用 401 指纹认出 DSH、改配置、重启桥接 | 一键恢复工具诞生 |

**踩坑**：`self-test.js` 默认写死 `:3080`，不读 config，直接跑会误报 `fetch failed`。

---

## 阶段六：给她人格与权限（17:40 – 18:20）

| 时间 | 动作 | 结果 |
|---|---|---|
| 17:45 | 新建人格卡 `roles/女儿.md`（1780 字） | 甜、软、偶尔毒舌、气质参照田熹微/章若楠 |
| 17:48 | `state/current-role.json` 设为该角色 | 中文被 PowerShell 5.1 按 ANSI 写坏（`女儿`→`濈冲効`），改用 `\u` 转义修好 |
| 17:52 | 自建 `mcp-qzone-safe.js`（独立 MCP server，不改原文件） | 空间 5 个工具：看动态/看说说/点赞/评论/发说说 |
| 17:56 | 发现 SnowLuma 有 9 个 qzone action 没接进 MCP | 这就是「空间权限没开」的真因 |
| 17:58 | 实测 `qzone_post` 发说说成功 | `tid=d1871ed63371e06a0dca0e00` |
| 18:00 | 修 `normalizeFeeds`：真实结构是 `data.msglist` + `time` + `comment_num` | 修好前「没有拉到说说」 |
| 18:02 | 她自主决定换名字/头像：**橘子汽水没气** / 「作业写不完就先睡吧」 | 头像描述：「淡蓝渐变，橘猫趴窗台打哈欠，半杯没气的橘子汽水」 |
| 18:05 | 按她的描述用 Pillow 画头像（512×512） | 猫、窗台、汽水杯、腮红、小星星 |
| 18:08 | 上传头像 + 改昵称/签名 | `set_qq_avatar` / `set_qq_profile` 都 retcode 0 |
| 18:15 | 用户报「昵称没变」 | `no_cache` 查三处接口，服务端确实已是新名 → 是客户端备注/缓存 |

---

## 阶段七：自主与约束（18:05 – 18:25）

| 时间 | 动作 | 结果 |
|---|---|---|
| 18:05 | 写 `autopilot.mjs`（自主时钟，90 分钟一轮） | 刷动态、点赞评论、偶尔发说说、偶尔主动私聊 |
| 18:10 | 桥接 proactive 调到 20–60 分钟 / 概率 0.4 | 她会主动找主人说话 |
| 18:10 | 写 `quota-watch.mjs`（每日 ¥5 警戒线，80% / 100% 两档提醒） | 每 5 分钟检查，QQ 私聊提醒 |
| 18:12 | 加 `persona_evolve`：她能往「成长笔记」写东西 | 她写的第一条：「爸爸喜欢我说话短一点」 |
| 18:20 | 导入 2 张种子表情（她的形象 + 橘猫头像） | `add_custom_face` 成功 |
| 18:22 | 加 `account_balance`：查余额 + 今日花费 | 余额 ¥24.31，够聊约 24 天（轻度） |

---

## 阶段八：开机自启与重启验证（18:25 – 18:35）

| 时间 | 动作 | 结果 |
|---|---|---|
| 18:25 | 用户重启电脑 | **自启链确实触发了**（自动拉起 DSH），但连踩三坑 |
| 18:26 | 坑 1：`QQNT` 被我当「非必要」禁了 | QQ 不在 → SnowLuma 无法注入 → 卡住 |
| 18:28 | 坑 2：`runtime.json` 里 `hookAutoLoad: false` | 改成 `true` 后立刻恢复：`[Hook] login detected` |
| 18:31 | 坑 3：桥接配置还是旧端口 58945 | 改成 55826 |
| 18:33 | 全部组件恢复 | 桥接日志：`机器人昵称: 橘子汽水没气`（**昵称确实改了**） |
| 18:35 | 给 `fix-dsh-port.ps1` 加 [0] 步：先确保 QQ 在跑 + 强制 `hookAutoLoad=true` | 下次重启可全自动 |

---

## 附：花掉的钱

| 项 | 金额 |
|---|---|
| DeepSeek 初始余额 | ¥9.74 |
| 用户充值 | +¥15 |
| 当前余额 | ¥24.31 |
| 今天（含大量调试）总花费 | 约 ¥0.11（高峰价估算） |
