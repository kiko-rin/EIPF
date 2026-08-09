# Glossary

| Term | Description |
|---|---|
| **EIPF** | Electric Interactive Publications Format, a ZIP-based format for interactive digital publications |
| **Series (.eipfs)** | Outermost container with `series.json`, renderer, shared engines and multiple Albums |
| **Album (.eipfa)** | Middle container with `album.json` and multiple Entries |
| **Entry (.eipf)** | Smallest content unit with `index.json`, `main.xml`, manifest and `body.xhtml` |
| **body.xhtml** | Content body of an Entry, carrying content as a linear `scene-entry` sequence |
| **scene-entry** | Content item types: dialog / Image / music / char / decision / sticker / controller, etc. |
| **char-slot** | Character slot inside a `char` item, with character ID, sprite path and canvas position ratios |
| **bgm-change** | Music change record independent of items |
| **manifest.xml** | `META-INF/manifest.xml` file manifest of an Entry, with sha256 checksums |
| **renderer/manifest.json** | Embedded renderer manifest; `type` is fixed to `embedded-renderer` |
| **embedded renderer** | A Web application shipped with EIPF that renders body.xhtml into an interactive page |
| **three-layer lookup** | Mechanism that resolves resources in Entry → Album → Series order |
| **IPC / postMessage protocol** | Communication protocol between reader and renderer (see `renderer-protocol.md`) |
| **bridge layer** | Script injected by the reader that forwards the renderer's `postMessage` to the host and exposes `__eipfInit/Open/Resource/Next/Prev/Config/State/Destroy` |
| **startIndex** | Resume target item index in `reader:open` (renderer `currentIndex`) |
| **low-refresh optimization (eink)** | Disables renderer typewriter and animations via `reader:init/config` to reduce screen refreshes |
| **degraded rendering mode** | Renders directly with `shared/scenario.js` / `.css` when no embedded renderer exists |
| **sharedResources** | Path configuration in series.json / album.json pointing to shared engines and the renderer |
| **l10n object** | Localized field, e.g., `{ "zh-CN": "..." }` |
| **formatVersion** | Specification version identifier, currently `"2.0.1-beta1"` (compatible with `"2.0.0"`) |
| **security** | Optional security attributes (signature / device binding / encryption), see `eipf-spec.md` §5 |
