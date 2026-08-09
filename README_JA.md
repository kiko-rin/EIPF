# EIPF — Electric Interactive Publications Format

> **EIPF v2.0.1-beta1** — 対話型デジタル出版物のためのオープンで IP 中立な仕様です。
> 中文 → [README.md](README.md) ／ English → [README_EN.md](README_EN.md)

![Spec](https://img.shields.io/badge/EIPF-v2.0.1--beta1-4C8CFF)
![License](https://img.shields.io/badge/license-CC0--1.0-blue)
![Docs](https://img.shields.io/badge/docs-zh--CN%20%7C%20en%20%7C%20ja-8A6BBE)

## ドキュメント（言語別）

| 言語 | ディレクトリ |
|---|---|
| 中文（中国語） | [`zh-CN/`](zh-CN/README.md) |
| English（英語） | [`en/`](en/README.md) |
| 日本語 | [`ja/`](ja/README.md) |
| JSON Schema（言語非依存） | [`schema/`](schema/README.md) |

## バージョン

| 項目 | 値 |
|---|---|
| `formatVersion` | `2.0.1-beta1` |
| テキストエンコーディング | UTF-8（BOM 禁止） |
| ZIP 暗号化・分割 | デフォルト禁止（任意のセキュリティ拡張を除く） |
| 名前空間 | `urn:eipf:spec:2.0`（main.xml）、`urn:eipf:manifest:2.0`（manifest.xml） |

## 概要

- **三層構造**：`Series (.eipfs)` → `Album (.eipfa)` → `Entry (.eipf)`、いずれも ZIP コンテナ。
- **フォーマット判定**：ルートの `series.json` / `album.json` / `index.json` で決定。
- **共有リソース**：Entry → Album → Series の順にカスケード検索。
- **コンテンツ**：各 Entry に `resource/text/body.xhtml`（XHTML シナリオ本文）を含む。
- **規範的コンテンツモデル**：body.xhtml の線形 `scene-entry` 列が唯一の規範コンテンツ。埋め込みレンダラーや共有スクリプトは任意の拡張。
- **IP 中立**：特定の知的財産に依存しない汎用フォーマット。
- **任意のセキュリティ**：2.0.1-beta1 以降、`signature` / `device` / `encryption` を著作権保護のためにサポート（デフォルト無効）。

## コントリビューション

- [貢献ガイド](CONTRIBUTING.md)（[中文](CONTRIBUTING.zh-CN.md) / [日本語](CONTRIBUTING.ja.md)）
- [行動規範](CODE_OF_CONDUCT.md) ・ [セキュリティポリシー](SECURITY.md) ・ [変更履歴](CHANGELOG.md)
- [ライセンス](LICENSE)（CC0 1.0 パブリックドメイン）
