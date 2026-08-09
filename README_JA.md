# EIPF — Electric Interactive Publications Format

> **EIPF v2.0.1-beta1** — 対話型デジタル出版物のためのオープンで IP 中立な仕様です。
> 中文 → [README.md](README.md) ／ English → [README_EN.md](README_EN.md)

![Spec](https://img.shields.io/badge/EIPF-v2.0.1--beta1-4C8CFF)
![License](https://img.shields.io/badge/license-CC0--1.0-blue)
![Docs](https://img.shields.io/badge/docs-zh--CN%20%7C%20en%20%7C%20ja-8A6BBE)

## はじめに

**EIPF（Electric Interactive Publications Format）** は、ZIP ベースの対話型デジタル出版物のためのオープンなフォーマット仕様です。以下を定義します：

- **三層の入れ子コンテナ構造**：`Series (.eipfs)` → `Album (.eipfa)` → `Entry (.eipf)`、いずれも ZIP コンテナ。
- **規範的コンテンツモデル**：各 Entry の `resource/text/body.xhtml` が線形の `scene-entry` 列でコンテンツを保持。
- **埋め込みレンダラープロトコル**：レンダラー（Web アプリ）とリーダー（ホスト）間の postMessage IPC。
- **任意のセキュリティ拡張**：`signature` / `device` / `encryption`（著作権保護用）。

本仕様は**汎用・IP 中立**です。特定の作品・ベンダー・知的財産に依存せず、各プロデューサは自作品の規約をここで定義される汎用構造にマッピングします。

ドキュメントは**中文 / English / 日本語**の三言語で管理されています。

## ドキュメント一覧

### コア仕様（spec）

| ドキュメント | 中文 | English | 日本語 |
|---|---|---|---|
| コア仕様：ZIP コンテナ、三層アーキテクチャ、JSON / XML 構造、任意のセキュリティ | [`zh-CN/spec/eipf-spec.md`](zh-CN/spec/eipf-spec.md) | [`en/spec/eipf-spec.md`](en/spec/eipf-spec.md) | [`ja/spec/eipf-spec.md`](ja/spec/eipf-spec.md) |
| コンテンツ形式：body.xhtml の構造と scene-entry 項目型 | [`zh-CN/spec/body-xhtml.md`](zh-CN/spec/body-xhtml.md) | [`en/spec/body-xhtml.md`](en/spec/body-xhtml.md) | [`ja/spec/body-xhtml.md`](ja/spec/body-xhtml.md) |
| 埋め込みレンダラーとリーダー間の IPC プロトコル | [`zh-CN/spec/renderer-protocol.md`](zh-CN/spec/renderer-protocol.md) | [`en/spec/renderer-protocol.md`](en/spec/renderer-protocol.md) | [`ja/spec/renderer-protocol.md`](ja/spec/renderer-protocol.md) |
| renderer/manifest.json のフィールド定義 | [`zh-CN/spec/renderer-manifest.md`](zh-CN/spec/renderer-manifest.md) | [`en/spec/renderer-manifest.md`](en/spec/renderer-manifest.md) | [`ja/spec/renderer-manifest.md`](ja/spec/renderer-manifest.md) |

### ガイドと用語（wiki）

| ドキュメント | 中文 | English | 日本語 |
|---|---|---|---|
| 制作ガイド：EIPF ファイルの生成方法 | [`zh-CN/wiki/producer-guide.md`](zh-CN/wiki/producer-guide.md) | [`en/wiki/producer-guide.md`](en/wiki/producer-guide.md) | [`ja/wiki/producer-guide.md`](ja/wiki/producer-guide.md) |
| リーダー統合ガイド：解析、カスケード検索、レンダラー読み込み | [`zh-CN/wiki/reader-integration.md`](zh-CN/wiki/reader-integration.md) | [`en/wiki/reader-integration.md`](en/wiki/reader-integration.md) | [`ja/wiki/reader-integration.md`](ja/wiki/reader-integration.md) |
| 用語集 | [`zh-CN/wiki/glossary.md`](zh-CN/wiki/glossary.md) | [`en/wiki/glossary.md`](en/wiki/glossary.md) | [`ja/wiki/glossary.md`](ja/wiki/glossary.md) |

### JSON Schema（言語非依存）

| Schema | ファイル |
|---|---|
| `series.json` — Series レイヤー | [`schema/series.schema.json`](schema/series.schema.json) |
| `album.json` — Album レイヤー | [`schema/album.schema.json`](schema/album.schema.json) |
| `index.json` — Entry レイヤー | [`schema/entry.schema.json`](schema/entry.schema.json) |
| `renderer/manifest.json` — 埋め込みレンダラー宣言 | [`schema/renderer-manifest.schema.json`](schema/renderer-manifest.schema.json) |

### 言語ディレクトリ

| 言語 | ディレクトリ |
|---|---|
| 中文（中国語） | [`zh-CN/`](zh-CN/README.md) |
| English（英語） | [`en/`](en/README.md) |
| 日本語 | [`ja/`](ja/README.md) |
| JSON Schema | [`schema/`](schema/README.md) |

## 概要

- **三層構造**：`Series (.eipfs)` → `Album (.eipfa)` → `Entry (.eipf)`、いずれも ZIP コンテナ。
- **フォーマット判定**：ルートの `series.json` / `album.json` / `index.json` で決定。
- **共有リソース**：Entry → Album → Series の順にカスケード検索。
- **コンテンツ**：各 Entry に `resource/text/body.xhtml`（XHTML シナリオ本文）を含む。
- **規範的コンテンツモデル**：body.xhtml の線形 `scene-entry` 列が唯一の規範コンテンツ。埋め込みレンダラーや共有スクリプトは任意の拡張。
- **IP 中立**：特定の知的財産に依存しない汎用フォーマット。
- **任意のセキュリティ**：2.0.1-beta1 以降、`signature` / `device` / `encryption` を著作権保護のためにサポート（デフォルト無効）。

## バージョン

| 項目 | 値 |
|---|---|
| `formatVersion` | `"2.0.1-beta1"`（`"2.0.0"` と互換） |
| テキストエンコーディング | UTF-8（BOM 禁止） |
| ZIP 暗号化・分割 | デフォルト禁止（任意のセキュリティ拡張を除く） |
| 名前空間 | `urn:eipf:spec:2.0`（main.xml）、`urn:eipf:manifest:2.0`（manifest.xml） |

## 読書順

```
eipf-spec.md → body-xhtml.md → renderer-manifest.md → renderer-protocol.md
→ producer-guide.md → reader-integration.md
```

## コントリビューション

- [貢献ガイド](CONTRIBUTING.md)（[中文](CONTRIBUTING.zh-CN.md) / [日本語](CONTRIBUTING.ja.md)）
- [行動規範](CODE_OF_CONDUCT.md) ・ [セキュリティポリシー](SECURITY.md) ・ [変更履歴](CHANGELOG.md)
- [引用情報](CITATION.cff) ・ [ライセンス](LICENSE)（CC0 1.0 パブリックドメイン）
