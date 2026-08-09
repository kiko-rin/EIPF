# 内嵌渲染器与阅读器 IPC 通信规范 v2.0.1-beta1

> 本规范描述**渲染器（renderer/ 中的 Web 应用）与阅读器（宿主）之间的通信**，
> 含消息格式、按键事件捕获、配置下发与资源请求。

## 1. 架构与传输

- 渲染器是 EIPF 内 `renderer/` 的 Web 应用（`index.html` + `engine.js` + `style.css` + `manifest.json`）。
- 阅读器把 `index.html` 内联引擎/样式后，载入 **WebView / iframe**。
- 传输：**`window.postMessage`**。渲染器以 `window.parent.postMessage(msg, '*')` 发送；阅读器侧注入**桥接层**捕获并转发到宿主。
- **阅读器是宿主，渲染器是子帧**；渲染器**不直接访问 ZIP/文件系统**，所有资源经 `renderer:resource` 请求、由阅读器响应。

### 1.1 桥接层（阅读器注入）

阅读器注入一段桥接脚本：
- 监听 `window` 的 `message`，把 `source === 'renderer'` 的消息转发给宿主。
- 提供 `window.__eipfInit / __eipfOpen / __eipfResource`，由宿主调用并向渲染器派发 `reader:*` 消息。
- 提供 `window.__eipfNext / __eipfPrev / __eipfConfig / __eipfState` 供按键/配置调用。

## 2. 渲染器查找（三级）

阅读器按 Entry → Album → Series 顺序查找 `renderer/manifest.json`：

| 优先级 | 位置 |
|---|---|
| 1 | Entry 文件 |
| 2 | Album 文件 |
| 3 | Series 文件 |

仅当 `manifest.type === "embedded-renderer"` 视为内嵌渲染器；三层均无则走降级渲染（共享脚本）。

## 3. 装载流程

1. 读取 `manifest.entry`（`index.html`）。
2. 内联 `manifest.engine`（`engine.js`）替换 `<script src="engine.js">`。
3. 内联 `manifest.style`（`style.css`）到 `</head>` 前。
4. 注入桥接脚本到 `</body>` 前。
5. 载入页面。
6. 页面加载完成后（`onPageFinished`）：发送 `reader:init`（含 `config.eink`）→ 发送 `reader:open`（含正文与 `startIndex`）。

## 4. 消息通用格式

```typescript
interface EIPFMessage {
  type: string;
  source: 'reader' | 'renderer';
  id?: string;      // 请求-响应匹配用
  payload?: any;
}
```

接收方**必须校验 `source`**，忽略非法来源。

## 5. 阅读器 → 渲染器（reader → renderer）

| 消息 | 时机 | payload |
|---|---|---|
| `reader:init` | 页面就绪 | `{ css, config: { eink } }` |
| `reader:config` | 运行中改配置 | `{ eink }` |
| `reader:open` | 打开章节 | `{ entryId, xhtml, metadata, startIndex }` |
| `reader:resource` | 响应资源请求 | `{ data, mime }` 或 `{ error, message }`（带 `id`） |
| `reader:destroy` | 关闭 | `{}` |

### reader:init

```json
{
  "type": "reader:init", "source": "reader",
  "payload": { "css": "", "config": { "eink": true } }
}
```

- `config.eink`：墨水屏/低刷新优化开关；`true` 时渲染器关闭打字机与过渡动画。

### reader:open

```json
{
  "type": "reader:open", "source": "reader", "id": "open_1",
  "payload": {
    "entryId": "entry_001",
    "xhtml": "<!DOCTYPE html>...body.xhtml 原文...",
    "metadata": {},
    "startIndex": 0
  }
}
```

- `xhtml`：Entry 的 `body.xhtml` 原文。
- `startIndex`：**续读目标条目索引**（渲染器 `currentIndex`）；`>0` 时渲染器直接重放到该条，否则从首条开始。

### reader:resource（响应）

```json
{ "type": "reader:resource", "source": "reader", "id": "res_0",
  "payload": { "data": "<ArrayBuffer>", "mime": "image/png" } }
```

失败：`{ "error": "not_found", "message": "..." }`。

### reader:config

```json
{ "type": "reader:config", "source": "reader", "payload": { "eink": false } }
```

运行时切换低刷新优化。

## 6. 渲染器 → 阅读器（renderer → reader）

| 消息 | 时机 | payload |
|---|---|---|
| `renderer:ready` | 初始化完成 | `{ version, capabilities }` |
| `renderer:rendered` | 章节渲染完成 | `{ entryId, dialogueCount, sceneCount, hasChoices }` |
| `renderer:state` | 每次推进/续读 | `{ currentIndex, totalEntries, type, progress }` |
| `renderer:resource` | 请求资源 | `{ path }`（带 `id`） |
| `renderer:ended` | 本章看完 | `{ entryId }` |
| `renderer:choice:selected` | 选项被选 | `{ index, value }` |
| `renderer:config:applied` | 配置生效 | `{ eink }` |
| `renderer:log` | 日志 | `{ level, message }` |
| `renderer:error` | 出错 | `{ code, message }` |
| `renderer:navigate` | 请求跳转 | `{ target, ctrlcmd }` |

### renderer:ready

```json
{ "type": "renderer:ready", "source": "renderer",
  "payload": { "version": "2.0.1-beta1", "capabilities": ["scenario","dialogue","typewriter","characters","backgrounds","choices","navigation","export"] } }
```

### renderer:state

```json
{ "type": "renderer:state", "source": "renderer",
  "payload": { "currentIndex": 5, "totalEntries": 120, "type": "dialogue", "progress": 0.0417 } }
```

阅读器据 `currentIndex` 保存**精确到条目**的进度。

### renderer:resource（请求）

```json
{ "type": "renderer:resource", "source": "renderer",
  "id": "res_0", "payload": { "path": "resource/backgrounds/bg.png" } }
```

阅读器经三级查找解析 `path`，以 `reader:resource` 响应（`data` 为资源字节，`mime` 由扩展名推断）。

### renderer:ended

```json
{ "type": "renderer:ended", "source": "renderer", "payload": { "entryId": "entry_001" } }
```

阅读器据此进入下一章。

### renderer:navigate

```json
{ "type": "renderer:navigate", "source": "renderer",
  "payload": { "target": "home", "ctrlcmd": "[GotoPage(dest=home)]" } }
```

渲染器请求跳转到其他页面；目标语义由阅读器实现。

## 7. 按键事件捕获规范

按键/触摸在**阅读器宿主层捕获**，再通过桥接层驱动渲染器，渲染器自身不监听物理键：

| 事件 | 阅读器处理 | 目标 API |
|---|---|---|
| 上一页 | 捕获 → 调用渲染器 | `window.__eipfPrev()` |
| 下一页 | 捕获 → 调用渲染器 | `window.__eipfNext()` |
| 屏幕中心轻点 | 阅读器拦截 → 呼出/隐藏导航栏（原生） | 不传给渲染器 |
| 点击对话框/正文 | 渲染器内部推进（自身 click） | 渲染器内处理 |

### 桥接控制 API（渲染器暴露）

| 全局函数 | 说明 |
|---|---|
| `window.__eipfNext()` | 下一页（跳过打字 → 推进 → 结尾触发 `renderer:ended`） |
| `window.__eipfPrev()` | 上一页（重放到前一对话） |
| `window.__eipfConfig(eink)` | 设置低刷新优化（发 `reader:config`） |
| `window.__eipfState()` | 返回 `{ currentIndex, totalEntries }`（供宿主查询） |
| `window.__eipfDestroy()` | 销毁渲染器（发 `reader:destroy`） |

## 8. 资源请求-响应时序

```
渲染器                         阅读器
  │ renderer:resource {id,path} │
  ├──────────────────────────────▶ 解析 path（Entry→Album→Series，必要时解密）
  │ reader:resource {id,data,mime}│
  ◀──────────────────────────────┤
  生成 Blob URL 渲染
```

## 9. 低刷新优化（eink）

- 阅读器在 `reader:init` 传 `config.eink`；运行中可用 `reader:config` 切换。
- 渲染器开启后：打字机直接整段显示；各层 `transition` 置为 `none`，减少屏幕刷新。
- 渲染器处理完发送 `renderer:config:applied`。

## 10. 生命周期

```
reader:init ─▶ reader:open(startIndex) ─▶ (推进/按键) ─▶ renderer:ended ─▶ 下一章
                                                └▶ reader:destroy（关闭）
```
