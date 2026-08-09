# renderer/manifest.json 字段规范 v2.0.1-beta1

> 内嵌渲染器清单文件，阅读器据此识别并装载渲染器。

## 1. 示例

```json
{
  "name": "Default EIPF Scenario Renderer",
  "type": "embedded-renderer",
  "version": "1.0.0",
  "entry": "index.html",
  "engine": "engine.js",
  "style": "style.css",
  "capabilities": ["scenario", "dialogue", "typewriter", "characters", "backgrounds", "choices", "navigation", "export"]
}
```

## 2. 字段说明

| 字段 | 类型 | 必需 | 说明 |
|---|---|---|---|
| `name` | string | ★ | 渲染器名称 |
| `type` | string | ★ | 固定 `"embedded-renderer"`（阅读器据此识别） |
| `version` | string | ★ | 渲染器版本 |
| `entry` | string | ★ | 入口 HTML 文件名（如 `index.html`） |
| `engine` | string | | 引擎 JS 文件名（如 `engine.js`），阅读器将其内联进 entry |
| `style` | string | | 样式文件名（如 `style.css`），阅读器将其内联进 entry |
| `capabilities` | string[] | | 渲染器能力列表 |

## 3. 阅读器使用方式

- `entry`：装载入口 HTML（WebView / iframe）。
- `engine`：将 `<script src="engine">` 替换为内联脚本；无精确匹配则移除外部 script 并在 `</body>` 前内联。
- `style`：在 `</head>` 前内联 `<style>`。
- `capabilities`：渲染器经 `renderer:ready` 上报，阅读器可据此展示能力。
- **IPC**：装载后阅读器注入桥接层，按 `renderer-protocol.md` 双向通信（`reader:init/open/config`、`renderer:state/ended` 等，含按键翻页 `__eipfNext/__eipfPrev` 与低刷新 `eink` 配置）。

## 4. 兼容性

- `formatVersion`、`communication`、`output`、`sandbox` 等字段为历史草案概念，**不作为必需字段**；阅读器应忽略未知字段。
