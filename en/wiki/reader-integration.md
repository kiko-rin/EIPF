# Reader Integration Guide

> An EIPF reader must cover the following capabilities. The reader is responsible for unpacking, layer parsing, resource lookup and renderer loading.

## 1. File Loading

1. Load the selected file with a ZIP library.
2. Build a `path -> content` map (skip directories).
3. Detect the layer: `series.json` > `album.json` > `index.json`, otherwise error.
4. Read the title: `meta.title` (l10n object, prefer the current language), fall back to the file name.

## 2. Three-Layer File Model

The reader maintains three file maps:

| Field | Layer | Description |
|---|---|---|
| Series files | Series | all files of the outer ZIP |
| Album files | Album | files of the currently loaded Album |
| Entry files | Entry | files of the currently loaded Entry |

### Nested ZIP Unpacking

- Load Album: read the entry at `album.path` from the Series layer and unpack it.
- Load Entry: look in the Album layer first, then the Entry ZIP in the Series layer; after unpacking, automatically strip a common prefix if all files share one.

## 3. Three-Layer Resource Lookup

Any resource is resolved in Entry → Album → Series order:

```
current Entry ZIP → current Album ZIP → Series ZIP → not found
```

On hit, convert to a Blob / Data URL and hand it to the rendering layer.

## 4. Renderer Loading

1. Look up `renderer/manifest.json` in three layers, requiring `type === "embedded-renderer"`.
2. Found → embedded renderer mode (see `renderer-protocol.md`).
3. Not found → degraded rendering mode (load `shared/scenario.js` / `scenario.css`).

## 5. Rendering a Chapter

1. Unpack the current Album and Entry.
2. Find `body.xhtml` (any file ending with `body.xhtml`).
3. Load the renderer or use degraded rendering.
4. Embedded mode: send `reader:open` (body resources replaced with locally accessible forms).
5. Degraded mode: assemble `<style>scenario.css</style>` + `<body>body.xhtml</body>` + `<script>scenario.js</script>` into a WebView / iframe.

## 6. Chapter List and Advancement

- List: expand `albums[]` in the Series layer; on tap, unpack the Album and read `album.json.entries[]`.
- Advancement: after `renderer:ended`, go to the next Entry in the same Album → next Album → end.

## 7. Communication Notes

- All messages carry a `source` field; the receiver must validate it.
- Resource request-response is matched by `id`.
- The renderer does not access the filesystem directly; resources are requested via `renderer:resource` and answered by the reader.
- Progress: save item-precise progress from `currentIndex` in `renderer:state`.

## 8. Optional Security (2.0.1-beta1)

When opening a file, if the root JSON contains the `security` attribute (see `eipf-spec.md` §5), perform checks per `enforcement`:

| enforcement | Reader action |
|---|---|
| `signature` | Verify `contentHash` with the embedded public key; reject on failure |
| `device` | Validate `security/license.json` (signature + device token match + not expired) |
| `encryption` | Decrypt resources before handing them to the rendering layer |

- If `enforcement` is unsupported, prompt "a newer reader is required"; do not silently downgrade.
- The device token should be derived from secure hardware (e.g., a non-exportable Android Keystore key) and shown in Settings for packager binding.

## 9. Key Constants

- Shared CSS path: `shared/scenario.css`.
- Shared JS path: `shared/scenario.js`.
- Body file: `resource/text/body.xhtml` (matched by suffix `body.xhtml`).
- Renderer entry: `renderer/<manifest.entry>`.
