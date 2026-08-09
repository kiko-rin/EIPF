# Embedded Renderer & Reader IPC Specification v2.0.1-beta1

> This specification describes **communication between the renderer (a Web application in `renderer/`) and the reader (host)**, including message formats, key-event capture, configuration and resource requests.

## 1. Architecture and Transport

- The renderer is a Web application in `renderer/` (`index.html` + `engine.js` + `style.css` + `manifest.json`).
- The reader inlines the engine/style into `index.html`, then loads it in a **WebView / iframe**.
- Transport: **`window.postMessage`**. The renderer sends via `window.parent.postMessage(msg, '*')`; the reader injects a **bridge layer** that captures and forwards messages to the host.
- **The reader is the host; the renderer is a child frame**. The renderer does **not** access the ZIP/filesystem directly; all resources are requested via `renderer:resource` and answered by the reader.

### 1.1 Bridge Layer (injected by the reader)

The reader injects a bridge script:
- Listens to `window` `message` events and forwards `source === 'renderer'` messages to the host.
- Exposes `window.__eipfInit / __eipfOpen / __eipfResource`, called by the host to dispatch `reader:*` messages to the renderer.
- Exposes `window.__eipfNext / __eipfPrev / __eipfConfig / __eipfState / __eipfDestroy` for key handling / configuration.

## 2. Renderer Lookup (three layers)

The reader looks up `renderer/manifest.json` in Entry → Album → Series order:

| Priority | Location |
|---|---|
| 1 | Entry files |
| 2 | Album files |
| 3 | Series files |

Only `manifest.type === "embedded-renderer"` counts as an embedded renderer; if none is found in all three layers, fall back to degraded rendering (shared scripts).

## 3. Loading Flow

1. Read `manifest.entry` (`index.html`).
2. Inline `manifest.engine` (`engine.js`) replacing `<script src="engine.js">`.
3. Inline `manifest.style` (`style.css`) before `</head>`.
4. Inject the bridge script before `</body>`.
5. Load the page.
6. After the page loads (`onPageFinished`): send `reader:init` (with `config.eink`) → send `reader:open` (with the body and `startIndex`).

## 4. Common Message Format

```typescript
interface EIPFMessage {
  type: string;
  source: 'reader' | 'renderer';
  id?: string;      // for request-response matching
  payload?: any;
}
```

The receiver **MUST validate `source`** and ignore messages from unknown origins.

## 5. Reader → Renderer (reader → renderer)

| Message | Timing | payload |
|---|---|---|
| `reader:init` | Page ready | `{ css, config: { eink } }` |
| `reader:config` | Runtime config change | `{ eink }` |
| `reader:open` | Open chapter | `{ entryId, xhtml, metadata, startIndex }` |
| `reader:resource` | Resource response | `{ data, mime }` or `{ error, message }` (with `id`) |
| `reader:destroy` | Close | `{}` |

### reader:init

```json
{
  "type": "reader:init", "source": "reader",
  "payload": { "css": "", "config": { "eink": true } }
}
```

- `config.eink`: e-ink / low-refresh optimization toggle; when `true`, the renderer disables typewriter and transition animations.

### reader:open

```json
{
  "type": "reader:open", "source": "reader", "id": "open_1",
  "payload": {
    "entryId": "entry_001",
    "xhtml": "<!DOCTYPE html>...body.xhtml raw...",
    "metadata": {},
    "startIndex": 0
  }
}
```

- `xhtml`: the Entry's raw `body.xhtml`.
- `startIndex`: **resume target item index** (renderer `currentIndex`); when `>0` the renderer replays up to that item, otherwise starts from the first.

### reader:resource (response)

```json
{ "type": "reader:resource", "source": "reader", "id": "res_0",
  "payload": { "data": "<ArrayBuffer>", "mime": "image/png" } }
```

On failure: `{ "error": "not_found", "message": "..." }`.

### reader:config

```json
{ "type": "reader:config", "source": "reader", "payload": { "eink": false } }
```

Switches the low-refresh optimization at runtime.

## 6. Renderer → Reader (renderer → reader)

| Message | Timing | payload |
|---|---|---|
| `renderer:ready` | Initialization complete | `{ version, capabilities }` |
| `renderer:rendered` | Chapter rendered | `{ entryId, dialogueCount, sceneCount, hasChoices }` |
| `renderer:state` | Each advance / resume | `{ currentIndex, totalEntries, type, progress }` |
| `renderer:resource` | Resource request | `{ path }` (with `id`) |
| `renderer:ended` | Chapter finished | `{ entryId }` |
| `renderer:choice:selected` | Option selected | `{ index, value }` |
| `renderer:config:applied` | Config applied | `{ eink }` |
| `renderer:log` | Log | `{ level, message }` |
| `renderer:error` | Error | `{ code, message }` |
| `renderer:navigate` | Navigation request | `{ target, ctrlcmd }` |

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

The reader saves **item-precise progress** from `currentIndex`.

### renderer:resource (request)

```json
{ "type": "renderer:resource", "source": "renderer",
  "id": "res_0", "payload": { "path": "resource/backgrounds/bg.png" } }
```

The reader resolves `path` by three-layer lookup and responds with `reader:resource` (`data` as resource bytes, `mime` inferred from the extension).

### renderer:ended

```json
{ "type": "renderer:ended", "source": "renderer", "payload": { "entryId": "entry_001" } }
```

The reader advances to the next chapter on this message.

### renderer:navigate

```json
{ "type": "renderer:navigate", "source": "renderer",
  "payload": { "target": "home", "ctrlcmd": "[GotoPage(dest=home)]" } }
```

The renderer requests navigation to another page; target semantics are implemented by the reader.

## 7. Key Event Capture

Key/touch events are captured **in the reader host layer** and drive the renderer through the bridge; the renderer itself does not listen to physical keys:

| Event | Reader handling | Target API |
|---|---|---|
| Previous page | Capture → call renderer | `window.__eipfPrev()` |
| Next page | Capture → call renderer | `window.__eipfNext()` |
| Center-screen tap | Reader intercepts → toggle nav bar (native) | not forwarded to renderer |
| Tap dialogue / body | Renderer advances internally (its own click) | handled in renderer |

### Bridge Control API (exposed by the renderer)

| Global function | Description |
|---|---|
| `window.__eipfNext()` | Next page (skip typing → advance → `renderer:ended` at end) |
| `window.__eipfPrev()` | Previous page (replay to previous dialogue) |
| `window.__eipfConfig(eink)` | Set low-refresh optimization (sends `reader:config`) |
| `window.__eipfState()` | Returns `{ currentIndex, totalEntries }` (for the host) |
| `window.__eipfDestroy()` | Destroy renderer (sends `reader:destroy`) |

## 8. Resource Request-Response Sequence

```
Renderer                        Reader
  │ renderer:resource {id,path} │
  ├──────────────────────────────▶ resolve path (Entry→Album→Series, decrypt if needed)
  │ reader:resource {id,data,mime}│
  ◀──────────────────────────────┤
  create Blob URL and render
```

## 9. Low-Refresh Optimization (eink)

- The reader sends `config.eink` in `reader:init`; it can switch at runtime with `reader:config`.
- When enabled, the renderer displays typewriter text in full and sets all layer `transition` to `none`, reducing screen refreshes.
- The renderer sends `renderer:config:applied` after applying.

## 10. Lifecycle

```
reader:init ─▶ reader:open(startIndex) ─▶ (advance/keys) ─▶ renderer:ended ─▶ next chapter
                                                └▶ reader:destroy (close)
```
