# EIPF — Electric Interactive Publications Format

> **EIPF v2.0.1-beta1** — 面向交互式数字出版物的开放格式规范（作品无关 / IP 中立）。
> English → [README_EN.md](README_EN.md) ／ 日本語 → [README_JA.md](README_JA.md)

![Spec](https://img.shields.io/badge/EIPF-v2.0.1--beta1-4C8CFF)
![License](https://img.shields.io/badge/license-CC0--1.0-blue)
![Docs](https://img.shields.io/badge/docs-zh--CN%20%7C%20en%20%7C%20ja-8A6BBE)

## 简介

**EIPF（Electric Interactive Publications Format）** 是一套基于 ZIP 的、面向交互式数字出版物的开放格式规范。它定义：

- **三层嵌套容器结构**：`Series (.eipfs)` → `Album (.eipfa)` → `Entry (.eipf)`，均为 ZIP 容器。
- **规范内容模型**：每个 Entry 的 `resource/text/body.xhtml` 以线性 `scene-entry` 条目承载内容。
- **内嵌渲染器协议**：渲染器（Web 应用）与阅读器（宿主）之间的 postMessage IPC 通信。
- **可选安全扩展**：`signature`（签名）/ `device`（设备绑定）/ `encryption`（加密），用于版权保护。

本规范为**通用格式**，不绑定任何具体作品、厂商或 IP；各作品专属的命名与约定由制作者自行映射到本规范定义的通用结构上。

文档以**中文 / English / 日本語**三语维护，见下方目录。

## 文档目录

### 核心规范（spec）

| 文档 | 中文 | English | 日本語 |
|---|---|---|---|
| 核心规范：ZIP 容器、三层架构、JSON / XML 结构、可选安全 | [`zh-CN/spec/eipf-spec.md`](zh-CN/spec/eipf-spec.md) | [`en/spec/eipf-spec.md`](en/spec/eipf-spec.md) | [`ja/spec/eipf-spec.md`](ja/spec/eipf-spec.md) |
| 内容格式：body.xhtml 结构与 scene-entry 条目类型 | [`zh-CN/spec/body-xhtml.md`](zh-CN/spec/body-xhtml.md) | [`en/spec/body-xhtml.md`](en/spec/body-xhtml.md) | [`ja/spec/body-xhtml.md`](ja/spec/body-xhtml.md) |
| 内嵌渲染器与阅读器 IPC 通信协议 | [`zh-CN/spec/renderer-protocol.md`](zh-CN/spec/renderer-protocol.md) | [`en/spec/renderer-protocol.md`](en/spec/renderer-protocol.md) | [`ja/spec/renderer-protocol.md`](ja/spec/renderer-protocol.md) |
| renderer/manifest.json 字段定义 | [`zh-CN/spec/renderer-manifest.md`](zh-CN/spec/renderer-manifest.md) | [`en/spec/renderer-manifest.md`](en/spec/renderer-manifest.md) | [`ja/spec/renderer-manifest.md`](ja/spec/renderer-manifest.md) |

### 指南与术语（wiki）

| 文档 | 中文 | English | 日本語 |
|---|---|---|---|
| 制作指南：如何生成 EIPF 文件 | [`zh-CN/wiki/producer-guide.md`](zh-CN/wiki/producer-guide.md) | [`en/wiki/producer-guide.md`](en/wiki/producer-guide.md) | [`ja/wiki/producer-guide.md`](ja/wiki/producer-guide.md) |
| 阅读器集成指南：解析、三级查找、渲染器装载 | [`zh-CN/wiki/reader-integration.md`](zh-CN/wiki/reader-integration.md) | [`en/wiki/reader-integration.md`](en/wiki/reader-integration.md) | [`ja/wiki/reader-integration.md`](ja/wiki/reader-integration.md) |
| 术语表 | [`zh-CN/wiki/glossary.md`](zh-CN/wiki/glossary.md) | [`en/wiki/glossary.md`](en/wiki/glossary.md) | [`ja/wiki/glossary.md`](ja/wiki/glossary.md) |

### JSON Schema（语言无关）

| Schema | 文件 |
|---|---|
| `series.json` — Series 层 | [`schema/series.schema.json`](schema/series.schema.json) |
| `album.json` — Album 层 | [`schema/album.schema.json`](schema/album.schema.json) |
| `index.json` — Entry 层 | [`schema/entry.schema.json`](schema/entry.schema.json) |
| `renderer/manifest.json` — 内嵌渲染器清单 | [`schema/renderer-manifest.schema.json`](schema/renderer-manifest.schema.json) |

### 语言目录

| 语言 | 目录 |
|---|---|
| 中文 | [`zh-CN/`](zh-CN/README.md) |
| English | [`en/`](en/README.md) |
| 日本語 | [`ja/`](ja/README.md) |
| JSON Schema | [`schema/`](schema/README.md) |

## 快速概览

- **三层嵌套**：`Series (.eipfs)` → `Album (.eipfa)` → `Entry (.eipf)`，均为 ZIP 容器。
- **格式识别**：读取根目录 `series.json` / `album.json` / `index.json` 判定层级。
- **资源共享**：资源按 Entry → Album → Series 三级级联查找。
- **内容承载**：每个 Entry 含 `resource/text/body.xhtml`（XHTML 场景正文）。
- **规范内容模型**：body.xhtml 的线性 `scene-entry` 序列是唯一规范内容；内嵌渲染器与共享脚本均为可选增强。
- **作品无关**：本规范为通用格式，不绑定任何具体 IP。
- **可选安全**：2.0.1-beta1 起支持 `signature` / `device` / `encryption`，默认关闭。

## 版本约束

| 项 | 值 |
|---|---|
| `formatVersion` | `"2.0.1-beta1"`（兼容 `"2.0.0"`） |
| 文本编码 | UTF-8（禁止 BOM） |
| ZIP 加密 / 分卷 | 默认禁止（可选安全扩展除外） |
| 命名空间 | `urn:eipf:spec:2.0`（main.xml）、`urn:eipf:manifest:2.0`（manifest.xml） |

## 阅读顺序

```
eipf-spec.md → body-xhtml.md → renderer-manifest.md → renderer-protocol.md
→ producer-guide.md → reader-integration.md
```

## 参与

- [贡献指南](CONTRIBUTING.md)（[中文](CONTRIBUTING.zh-CN.md) / [日本語](CONTRIBUTING.ja.md)）
- [行为准则](CODE_OF_CONDUCT.md) · [安全策略](SECURITY.md) · [更新日志](CHANGELOG.md)
- [引用信息](CITATION.cff) · [许可协议](LICENSE)（CC0 1.0 公有领域）
