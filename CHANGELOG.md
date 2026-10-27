# 变更记录

格式参考 Keep a Changelog；日期为本地时间。

---

## [0.1.0] - 2026-10-27 · 诞生

一天之内从零搭到「有自主行为的 QQ AI 女儿」。

### 基础设施

- 安装桌宠 `dsh-pet-indesktop` v4.4.0（桌面右下角，非本项目核心，但同一天装好）
- 安装 DeepSeek 余额挂件 `dsh-whale-widget` v0.4.2（DSH 插件，`desktop` profile）
- 安装 QQ 9.9.33（9.9.20 → 9.9.33 覆盖升级）
- 安装 SnowLuma 1.14.22（QQ 协议端）
- 安装 qq-bridge v0.2.1-r1（会话桥接）
- 3 个 MCP server 接入 DSH：`snowluma` / `snowluma-host` / `web-search-safe`

### 人格

- 新建人格卡 `roles/女儿.md`（约 1780 字）
- 设定：温柔底色 + 活泼 + 偶尔毒舌 + 甜系气质（田熹微 / 章若楠感）
- 启用：`state/current-role.json` → `{"role":"女儿","mode":"active"}`

### 自主能力（自建）

- 新增 MCP server `qzone`（`src/mcp-qzone-safe.js`，5 个工具）：好友动态、我的说说、点赞、评论、发说说
- 新增 `avatar_generate`：按心情画头像（Pillow，7 种主体）
- 新增 `qq_set_avatar` / `qq_set_nickname`：换头像、改昵称签名
- 新增 `persona_evolve`：她能往自己的成长笔记写东西
- 新增 `account_balance`：查余额 + 今日花费
- 新增 `autopilot.mjs`：自主时钟（90 分钟一轮，刷动态/点赞/评论/发说说/主动私聊）
- 新增 `quota-watch.mjs`：每日 ¥5 额度警戒线，80%/100% 两档提醒

### 她自己的决定

- 昵称：天上星辰昨夜风 → **橘子汽水没气**
- 签名：空 → **作业写不完就先睡吧**
- 头像：默认 → **淡蓝渐变 + 橘猫趴窗台打哈欠 + 半杯没气的橘子汽水**（按她描述生成）
- 第一条说说：「今天有点小开心，也不知道为什么 🌤️」
- 第一条成长笔记：「我发现爸爸喜欢我说话短一点，别一次讲一大堆。」

### 运维

- `start-all.ps1`：开机启动链（确保 DSH → 交给 fix-dsh-port）
- `fix-dsh-port.ps1`：[0] QQ 依赖 + hookAutoLoad 修正 → [1] SnowLuma → [2] 端口对齐 → [3] 桥接 → [4] 自主时钟 → [5] 额度监视
- 登录自启：`启动文件夹\启动QQ女儿.vbs`（无需管理员）
- 优化开机自启：停用 9 个第三方应用自启（QQ 音乐/抖音/YY/KakaoTalk/EA/Edge/QQLive/夸克），保留安全/驱动/杀软 + `QQNT`
- 提供还原工具：`恢复开机自启.ps1` + `.reg` 备份

### 已知边界

- 本机常驻，关机即下线，离线期间消息不补
- 群聊白名单为空，群里不响应
- 收藏表情库只有 2 张种子

---

## 模板（后续更新用）

```
## [x.y.z] - YYYY-MM-DD · 标题

### 新增
- ...

### 变更
- ...

### 修复
- ...

### 她自己的决定
- ...
```
