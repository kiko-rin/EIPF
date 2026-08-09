# schema — JSON Schema 定义

> EIPF 各层元数据的 JSON Schema（draft-07）。
> 其他语言：English → [README_EN.md](README_EN.md) ／ 日本語 → [README_JA.md](README_JA.md)

| 文件 | 说明 |
|---|---|
| [`series.schema.json`](series.schema.json) | `series.json` — Series 层 |
| [`album.schema.json`](album.schema.json) | `album.json` — Album 层 |
| [`entry.schema.json`](entry.schema.json) | `index.json` — Entry 层 |
| [`renderer-manifest.schema.json`](renderer-manifest.schema.json) | `renderer/manifest.json` — 内嵌渲染器清单 |

- `formatVersion` 允许 `"2.0.0"` 与 `"2.0.1-beta1"`。
- `security` 属性为可选（2.0.1-beta1 新增），见 [`../zh-CN/spec/eipf-spec.md`](../zh-CN/spec/eipf-spec.md) §5。

> 规范文档按语言存放：中文 [`../zh-CN/`](../zh-CN/README.md) ／ English [`../en/`](../en/README.md) ／ 日本語 [`../ja/`](../ja/README.md)
