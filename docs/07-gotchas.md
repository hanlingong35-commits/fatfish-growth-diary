# 踩过的坑（按血泪程度排序）

这篇是整个日记里最值钱的部分。每一条都是真实卡住过、并且查到了根因的。

---

## 1. 桌面端不读 `web` profile —— 装插件装了个寂寞

**症状**：按插件 README 的 `dsh plugin --profile web add ...` 装完，重启啥也没有，控制台也不报错。

**根因**：DSH 桌面端读的是 **`desktop` profile**（`~/.dsh/profiles/desktop`），而 CLI 又拒绝 `--profile desktop`（"managed exclusively by the Electron application"）。

**解法**：`dsh plugin --profile desktop add ...` —— 这个命令是允许的（只有"启动"被拒，安装不拒）。

---

## 2. SnowLuma 依赖 QQ 客户端，别把 QQ 当"非必要自启"关掉

**症状**：重启电脑后 SnowLuma 停在 `SnowLuma starting`，3001 永远不通。

**根因**：SnowLuma 靠**注入 QQ 进程**工作。QQ 没起 = 没有注入目标。而我在优化自启时把 `QQNT` 一起禁了。

**解法**：`QQNT` 必须归为"她的必要组件"，恢复自启。

---

## 3. `hookAutoLoad: false` 会让 SnowLuma 静默不注入

**症状**：QQ 明明在跑，SnowLuma 也启动成功（5099 通），但就是没有 OneBot 端口，日志停在 `listening`。

**根因**：`SnowLuma/config/runtime.json` 里 `"hookAutoLoad": false`。

**解法**：改成 `true`。改完立刻出现：
```
[Hook] login detected: PID=17124 UIN=3592325073
[OneBot.HTTP] listening 127.0.0.1:3000
```
并且已经写进 `fix-dsh-port.ps1`，每次启动自动修正。

---

## 4. DSH 端口每次重启都变，桥接会失联

**症状**：重启 DSH 后 QQ 发消息没回应。

**根因**：DSH 桌面端端口依次是 56288 → 64303 → 58945 → 55826，**每次随机**。桥接配置写死端口。

**解法**：`fix-dsh-port.ps1` 扫本地监听端口，用 DSH 特有的指纹认人：
```
GET / → 401 + body「dsh web authentication required」
```

---

## 5. DSH 沙箱拦掉所有 HTTPS（443）

**症状**：`curl https://api.github.com` 返回 `exit 35`，连 baidu 都握手失败；但明文 HTTP 正常。

**根因**：沙箱的网络层掐 TLS。

**解法**：所有下载/API 调用走提权通道。这解释了为什么项目里一堆命令都需要提权。

---

## 6. PowerShell 5.1 会把 UTF-8 无 BOM 的脚本按 GBK 读

**症状**：`.ps1` 里的中文变成乱码，导致引号配对错乱、语法报错。

**解法**：写 `.ps1` 必须用 **UTF-8 with BOM**。中文参数写进 JSON 时用 `\uXXXX` 转义（`女儿` → `\u5973\u513F`）。

---

## 7. 命令行传 JSON 给 Python，引号会被吞

**症状**：`python gen-avatar.py '{"motif":"cat"}'` 生成的图用的是默认配色。

**根因**：PowerShell 调用 native 程序时引号被剥离，Python 收到非法 JSON → 回退默认值。

**解法**：改用 stdin 传参（脚本已支持 `-` 作为参数表示读 stdin）。

---

## 8. `self-test.js` 写死 `:3080`，不读 config

**症状**：链路明明是通的，跑自检报 `fetch failed`。

**根因**：`src/self-test.js` 第 39 行 `process.argv[2] ?? 'http://127.0.0.1:3080'`。

**解法**：手动带地址：`node src/self-test.js http://127.0.0.1:55826`。

---

## 9. 桥接退出后 SnowLuma 也容易跟着停

**症状**：重启 DSH 后桥接日志停在 `DSH 事件流已结束`。

**解法**：这正是 `fix-dsh-port.ps1` 存在的意义——它会检查每个组件并按需重启。

---

## 10. 昵称"没变"其实是客户端缓存 / 好友备注

**症状**：`set_qq_profile` 返回 retcode 0，`get_login_info` 却还是旧名。

**根因**：
- `get_login_info` 返回的是**登录时的缓存**
- QQ 好友列表显示的是**你给这个号设的备注**

**解法**：用 `get_stranger_info`（带 `no_cache`）或 `get_group_member_info` 查真实值。

---

## 11. 150% DPI 下截图抓不到分层窗口

**症状**：截图里看不到桌宠，以为没渲染。

**根因**：进程 DPI 不感知时，`CopyFromScreen` 拿的是缩放后的虚拟屏幕，且 GDI 默认不抓 `WS_EX_LAYERED` 窗口。

**解法**：`SetProcessDPIAware()` + `BitBlt` 带 `CAPTUREBLT (0x40000000)`。

---

## 12. 腾讯下载 CDN 对海外 IP 返回 403

**症状**：`qqdl.gtimg.cn` 全是 403，`dldir1.qq.com` 全是 404，连官网页面自己挂的链接都 404。

**根因**：出口 IP 在韩国（Korea Telecom），腾讯 CDN 按地域拦。

**解法**：用 GitHub 镜像下载，然后**验 Authenticode 签名**（签名伪造不了，验过就等于验过原包）。

---

## 13. `launcher.bat` / `start.bat` 用裸 `node` 会闪退

**症状**：双击没反应。

**根因**：脚本写的是 `node xxx.js`，依赖 PATH，而 PATH 里没有 Node。

**解法**：改成自动探测（PATH → SnowLuma 自带 → DSH 运行时）。

---

## 14. 多个同秒生成的文件会互相覆盖

**症状**：批量生成头像，三张图变成一个。

**解法**：文件名加随机后缀。

---

## 15. 清进程时别把自己的命令行也匹配进去

**症状**：一条清理命令执行到一半自杀（命令行里含 `fix-dsh-port` 关键字，匹配到了自己）。

**解法**：过滤时排除当前 PID，或者用更精确的匹配条件。
