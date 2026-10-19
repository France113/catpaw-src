# 猫爪复刻 · 弹幕源包（自研加密版）

**在线地址（直接填进 App 源地址）**

```
https://catpaw-danmu-src.app.workbuddy.host/index.js.md5
```

当前版本：`index.js` 1,255,166 字节，md5 `3cef038376f3a62a4aba6a5a6ef30cc4`

## 关于「源是不是加密了」

**没有。**（2026-10-19 实证）

- 官方 IPA 内的 `App.framework/flutter_assets/asset/js/index.js` 是**明文** JS，无自解压、无 eval 加密；
- 官方引擎里 `md5` 出现 **0 次**，`.md5` 文件只是**完整性校验**，不是解密密钥；
- 我们的源包在本地按真机同样的契约加载，`/check`、`/config` 全部 200，站点列表正常返回。

所以「拉下来用不了」的原因不在加密层，而在**采集站本身**（详见下节）。

## 2026-10-19 排查结论：站点大半已废

| 站点 | 状态 | 说明 |
|---|---|---|
| 非凡采集 ffm3u8 | ✅ 可用 | 首页 27 分类、`ac=detail&t=6` 返回 4572 条、详情正常 |
| 快看影视 kkys | ❌ 已废 | 主域 `api1.baibaipei.com` **DNS 已不存在**（域名停用） |
| 琨娱七七 kunyu77 | ❌ 已废 | `api.tyun77.cn` 被 **WAF 拦截**（返回拦截页） |
| 推送 push | — | type 4，没有 home/category，属正常 |

**结论：旧版源包在 App 里点开大概率看不到内容，因为 3 个视频站里有 2 个已下线。**

## 本版修复了什么

1. **请求头加固**（`src/util/req.js`）：统一注入移动端浏览器 UA + Accept + Accept-Language。
   部分采集站（如 `cj.ffzyapi.com`）对 Node/axios 默认请求头直接 403，加 UA 后正常 200。
2. **重新构建源包**：自研加密（AES/MD5，对照 crypto-js 字节等价）+ 弹幕引擎 v3 合体，
   md5 同步更新。

## 弹幕机制（无需重装 IPA）

弹幕做在**源层**，不依赖原生 dylib：

- 启动时：装 `globalThis.messageToDart` 垫片、包装 `catServerFactory` 截获 `/detail`（片名）与 `/play`（集数+地址）；
- 播放时：把 `url` 改写为 `127.0.0.1:52999/player.html?...`、`parse=0` 强制 WebView 打开；
- 同进程 sidecar 提供播放页 + 弹幕叠加（滚动/顶/底、颜色、字号）+ `/proxy/video` 视频代理（兼容 m3u8）；
- 弹幕来源：优先源内 `/danmu/auto`，失败回退 `https://dm.11031.xyz/87654321`。

IPA 侧只需 `NSAllowsLocalNetworking`（已开），所以**沿用已装的「猫爪复刻测试」IPA 即可**。

## 排错顺序

1. 先确认源地址是 `.../index.js.md5`（不是 `index.js`）；
2. 点「非凡采集」进分类，看能不能列出片名 —— 这是当前唯一可用的视频站；
3. 播放后看日志有没有 `[danmu]` 行：
   - 无任何 `[danmu]` → 源没加载（查 md5 是否一致、URL 可达）；
   - 有 `[danmu] 捕获播放` 但无弹幕 → 片名/集数没匹配（把日志发我调匹配）。

## 后续可做（等你定）

- **补可用视频站**：继续加可用的采集站 API（当前只剩 1 个，太单薄）；
- 把 maccms 类站点做成通用 type3 spider，一处配置多处复用。

---

发布管理：**前往「设置—数据管理—应用」中管理**。
