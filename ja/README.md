# EIPF ドキュメント（日本語）

> **EIPF v2.0.1-beta1** — 日本語ドキュメント。
> 中文 → [`../zh-CN/`](../zh-CN/README.md) ／ English → [`../en/`](../en/README.md)

## 目次

### spec — 仕様書

| ファイル | 内容 |
|---|---|
| [`spec/eipf-spec.md`](spec/eipf-spec.md) | コア仕様：ZIP コンテナ、三層アーキテクチャ、JSON / XML 構造、任意のセキュリティ |
| [`spec/body-xhtml.md`](spec/body-xhtml.md) | コンテンツ形式：body.xhtml の構造と scene-entry 項目型 |
| [`spec/renderer-protocol.md`](spec/renderer-protocol.md) | 埋め込みレンダラーとリーダー間の IPC プロトコル |
| [`spec/renderer-manifest.md`](spec/renderer-manifest.md) | renderer/manifest.json のフィールド定義 |

### wiki — ガイドと用語

| ファイル | 内容 |
|---|---|
| [`wiki/producer-guide.md`](wiki/producer-guide.md) | 制作ガイド：EIPF ファイルの生成方法 |
| [`wiki/reader-integration.md`](wiki/reader-integration.md) | リーダー統合ガイド |
| [`wiki/glossary.md`](wiki/glossary.md) | 用語集 |

### schema（言語非依存）

JSON Schema は [`../schema/`](../schema/README.md) にあります（series / album / entry / renderer-manifest）。

## 読書順

`spec/eipf-spec.md` → `spec/body-xhtml.md` → `wiki/producer-guide.md` → `wiki/reader-integration.md`。
