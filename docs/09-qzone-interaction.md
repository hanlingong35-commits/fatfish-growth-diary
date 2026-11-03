# QQ 空间评论互动：绕过上游限制的完整方案

> 2026-11-03 实现。这篇记录了「为什么做不到 → 怎么绕过去 → 踩了什么坑」的全过程。

## 问题

主人在她的说说下面留言，她一直没回应。

## 第一层诊断：功能压根不存在

查 `_snowluma-catalog.json` 里全部 190 个 action，qzone 相关只有 9 个：

```
comment_qzone        写评论
delete_qzone_msg     删说说
get_qzone_feeds      好友动态
get_qzone_msg_list   说说列表（含 comment_num）
like_qzone           点赞
send_qzone_msg       发说说
set_qzone_ban        拉黑
set_qzone_msg_right  改可见范围
unlike_qzone         取消赞
```

**只有写评论，没有读评论。** 所以她连「有人评论了」都不知道，更别说内容。

SnowLuma 源码里其实有 `getQzoneMsgDetail()`（调 `emotion_cgi_msgdetail_v6`，返回含 commentlist），但**没有暴露成 action**。

## 第二层尝试：用 cookie 直连 CGI（失败）

SnowLuma 有 `get_cookies` action，可以拿到 qzone 的 cookie。试着用它直连：

```js
GET https://h5.qzone.qq.com/proxy/domain/taotao.qq.com/cgi-bin/emotion_cgi_msgdetail_v6
  ?tid=<tid>&uin=<uin>&t1_source=1&not_trunc_con=1&need_right=1&not_adapt_outpic=1&g_tk=<bkn>
Cookie: <cookie>
```

`bkn = hash33(skey)`，按 SnowLuma 的实现复刻。**四次尝试全部 403。**

原因：`get_cookies` 导出的 cookie 不完整（缺 `p_skey` 等），openresty 网关直接拒绝。

## 第三层：浏览器自动化（成功）

**思路**：让真实浏览器登录空间，通过 CDP 在页面内读 DOM。浏览器天然带完整 cookie 和页面内签名。

### 架构

```
独立 Edge（--remote-debugging-port=9223，专用 profile）
   ↓ CDP Runtime.evaluate
穿透 #app_canvas_frame iframe
   ↓
li.feed[data-tid] → li.comments_item → .comments_content
   ↓ 解析「作者 : 内容」
qzone-browser.mjs → autopilot → 生成回复 → comment_qzone 发送
```

### 实现要点

**1. 页面选择**

`--app=https://user.qzone.qq.com/<uin>` 打开的是「主页」tab，**没有 iframe**。
必须导航到 **说说列表页 `/311`**：

```js
await send('Page.navigate', { url: 'https://user.qzone.qq.com/3592325073/311' });
```

**2. 等待渲染（关键）**

iframe 1 秒就存在，但说说要 5–6 秒才渲染出来。轮询条件必须是「说说条目出现」：

```js
const READY = `(() => {
  const f = document.getElementById('app_canvas_frame');
  try {
    const d = f.contentDocument;
    return !!(d && d.querySelectorAll('li.feed[data-tid]').length > 0);
  } catch (e) { return false; }
})()`;
```

**3. DOM 结构（实测）**

| 元素 | 选择器 | 说明 |
|---|---|---|
| 说说 | `li.feed[data-tid][data-uin]` | tid 在这里 |
| 评论项 | `.comments_item` | 每条评论 |
| 评论内容 | `.comments_content` | 文本形如「静水悠悠 : 不错喔，谁给你买的热可可」 |
| 评论时间 | `.comments_op` | 「2026-10-27回复」 |

解析要点：`.comments_content` 的 innerText **包含时间行**，必须只取第一行再按 `:` 拆分，否则内容里会混进「2026-10-27回复」。

**4. 刷新才能看到新评论**

DOM 是加载时的快照。每次读之前必须重新导航/刷新，否则读到的是旧数据。

## 踩的三个坑

### 坑 1：`ws` 包不存在

pnpm 的 node_modules 里没有直接暴露 `ws`。

**解决**：Node 24 有全局 `WebSocket`（WHATWG 标准），直接用，事件用 `addEventListener`。

### 坑 2：`--app` 模式没有说说 iframe

详见上面「页面选择」。**必须用 `/311`**。

### 坑 3：她把自己的回复当成新评论，无限自我对话

第一版只按 `tid + 作者 + 内容` 去重，结果：

```
静水悠悠   : 不错喔，谁给你买的热可可
橘子汽水没气: 爸爸买的呀，可惜没你的份～      ← 第一次回复
橘子汽水没气: 哼，我爸爸买给我的，你想得美啦～  ← 把自己的回复又回了一遍
```

**修复**：读评论前取一次自己的昵称，跳过作者是自己的评论。

```js
const selfNick = (await onebot('get_login_info'))?.nickname ?? '';
...
if (!c.author || !c.text || (selfNick && c.author === selfNick)) continue;
```

## 最终效果

```
14:00:34 [autopilot] 回复评论 静水悠悠：「不错喔，谁给你买的热可可」
                   →「爸爸买的呀，可惜没你的份～」
```

服务端确认：评论数 1 → 2。

## 依赖与降级

| 依赖 | 说明 |
|---|---|
| Edge 实例 | 专用 profile `state/edge-qzone`，登录态持久化（重启后仍有效） |
| 开机自启 | `fix-dsh-port.ps1` 的 `[0/6]` 步，实测杀掉后 21.3 秒自动拉起 |
| 降级 | 浏览器不可用时，退回「从 OneBot 拿评论数，私聊提醒主人」 |

## 遗留

- 只读「自己说说」的评论；读好友说说的评论需要另加导航逻辑
- 只支持文字评论的解析（图片评论、@ 会带噪声）
- 页面改版会导致选择器失效，需要维护
