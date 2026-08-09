# EIPF 规范文档集（中文）

> **EIPF v2.0.1-beta1** — 中文文档。
> English → [`../en/`](../en/README.md) ／ 日本語 → [`../ja/`](../ja/README.md)

## 目录

### spec — 规范文档

| 文件 | 内容 |
|---|---|
| [`spec/eipf-spec.md`](spec/eipf-spec.md) | 核心规范：ZIP 容器、三层架构、JSON / XML 结构、可选安全属性 |
| [`spec/body-xhtml.md`](spec/body-xhtml.md) | 内容格式：body.xhtml 结构与 scene-entry 条目类型 |
| [`spec/renderer-protocol.md`](spec/renderer-protocol.md) | 内嵌渲染器与阅读器 IPC 通信协议 |
| [`spec/renderer-manifest.md`](spec/renderer-manifest.md) | renderer/manifest.json 字段定义 |

### wiki — 指南与术语

| 文件 | 内容 |
|---|---|
| [`wiki/producer-guide.md`](wiki/producer-guide.md) | 制作指南：如何生成 EIPF 文件 |
| [`wiki/reader-integration.md`](wiki/reader-integration.md) | 阅读器集成指南 |
| [`wiki/glossary.md`](wiki/glossary.md) | 术语表 |

### schema（语言无关）

JSON Schema 位于 [`../schema/`](../schema/README.md)（series / album / entry / renderer-manifest）。

## 阅读顺序

`spec/eipf-spec.md` → `spec/body-xhtml.md` → `wiki/producer-guide.md` → `wiki/reader-integration.md`。
