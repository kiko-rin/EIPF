# renderer/manifest.json Field Specification v2.0.1-beta1

> Manifest of the embedded renderer; the reader uses it to detect and load the renderer.

## 1. Example

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

## 2. Field Reference

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | ★ | Renderer name |
| `type` | string | ★ | Fixed `"embedded-renderer"` (used by readers for detection) |
| `version` | string | ★ | Renderer version |
| `entry` | string | ★ | Entry HTML file name (e.g., `index.html`) |
| `engine` | string | | Engine JS file name (e.g., `engine.js`); the reader inlines it into entry |
| `style` | string | | Style file name (e.g., `style.css`); the reader inlines it into entry |
| `capabilities` | string[] | | Renderer capability list |

## 3. Reader Usage

- `entry`: loads the entry HTML (WebView / iframe).
- `engine`: replaces `<script src="engine">` with an inline script; if there is no exact match, removes the external script and inlines it before `</body>`.
- `style`: inlines a `<style>` block before `</head>`.
- `capabilities`: reported by the renderer via `renderer:ready`; the reader may use it to surface capabilities.
- **IPC**: after loading, the reader injects the bridge layer and communicates bidirectionally per `renderer-protocol.md` (`reader:init/open/config`, `renderer:state/ended`, etc., including key paging `__eipfNext/__eipfPrev` and low-refresh `eink` configuration).

## 4. Compatibility

- `formatVersion`, `communication`, `output`, `sandbox` are historical draft concepts and are **not required**; readers should ignore unknown fields.
