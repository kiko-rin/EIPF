# schema — JSON Schema 定義

> 各 EIPF レイヤーのメタデータのための JSON Schema（draft-07）。
> その他の言語：中文 → [README.md](README.md) ／ English → [README_EN.md](README_EN.md)

| ファイル | 内容 |
|---|---|
| [`series.schema.json`](series.schema.json) | `series.json` — Series レイヤー |
| [`album.schema.json`](album.schema.json) | `album.json` — Album レイヤー |
| [`entry.schema.json`](entry.schema.json) | `index.json` — Entry レイヤー |
| [`renderer-manifest.schema.json`](renderer-manifest.schema.json) | `renderer/manifest.json` — 埋め込みレンダラー宣言 |

- `formatVersion` は `"2.0.0"` と `"2.0.1-beta1"` を許容。
- `security` プロパティは任意（2.0.1-beta1 で追加）、[`../ja/spec/eipf-spec.md`](../ja/spec/eipf-spec.md) §5 を参照。

> 仕様書は言語別に配置：中文 [`../zh-CN/`](../zh-CN/README.md) ／ English [`../en/`](../en/README.md) ／ 日本語 [`../ja/`](../ja/README.md)
