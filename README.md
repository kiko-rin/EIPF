# EIPF — Electric Interactive Publications Format

> **EIPF v2.0.1-beta1** — 面向交互式数字出版物的开放格式规范（作品无关 / IP 中立）。
> English → [README_EN.md](README_EN.md) ／ 日本語 → [README_JA.md](README_JA.md)

![Spec](https://img.shields.io/badge/EIPF-v2.0.1--beta1-4C8CFF)
![License](https://img.shields.io/badge/license-CC0--1.0-blue)
![Docs](https://img.shields.io/badge/docs-zh--CN%20%7C%20en%20%7C%20ja-8A6BBE)

## 文档（按语言）

| 语言 | 目录 |
|---|---|
| 中文 | [`zh-CN/`](zh-CN/README.md) |
| English | [`en/`](en/README.md) |
| 日本語 | [`ja/`](ja/README.md) |
| JSON Schema（语言无关） | [`schema/`](schema/README.md) |

## 版本

| 项 | 值 |
|---|---|
| `formatVersion` | `2.0.1-beta1` |
| 文本编码 | UTF-8（禁止 BOM） |
| ZIP 加密 / 分卷 | 默认禁止（可选安全扩展除外） |
| 命名空间 | `urn:eipf:spec:2.0`（main.xml）、`urn:eipf:manifest:2.0`（manifest.xml） |

## 快速概览

- **三层嵌套**：`Series (.eipfs)` → `Album (.eipfa)` → `Entry (.eipf)`，均为 ZIP 容器。
- **格式识别**：读取根目录 `series.json` / `album.json` / `index.json` 判定层级。
- **资源共享**：资源按 Entry → Album → Series 三级级联查找。
- **内容承载**：每个 Entry 含 `resource/text/body.xhtml`（XHTML 场景正文）。
- **规范内容模型**：body.xhtml 的线性 `scene-entry` 序列是唯一规范内容；内嵌渲染器与共享脚本均为可选增强。
- **作品无关**：本规范为通用格式，不绑定任何具体 IP。
- **可选安全**：2.0.1-beta1 起支持 `signature`（签名）/ `device`（设备绑定）/ `encryption`（加密），默认关闭。

## 参与

- [贡献指南](CONTRIBUTING.md)（[中文](CONTRIBUTING.zh-CN.md) / [日本語](CONTRIBUTING.ja.md)）
- [行为准则](CODE_OF_CONDUCT.md) · [安全策略](SECURITY.md) · [更新日志](CHANGELOG.md)
- [许可协议](LICENSE)（CC0 1.0 公有领域）
