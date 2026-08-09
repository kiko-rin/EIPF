# EIPF への貢献

**EIPF（Electric Interactive Publications Format）** への貢献にご関心をお寄せいただき、ありがとうございます。

このリポジトリは**ドキュメント・仕様プロジェクト**です。規範的な Markdown ドキュメントと JSON Schema が含まれます。issue または pull request を開く前に、このガイドをお読みください。

> 中文: [CONTRIBUTING.zh-CN.md](CONTRIBUTING.zh-CN.md) ／ English: [CONTRIBUTING.md](CONTRIBUTING.md)

## 貢献の方法

- **バグ報告**：ドキュメントまたは Schema の誤り（不正な例、リンク切れ、曖昧な表現）。
- **仕様変更の提案**：新フィールド、新しい `scene-entry` 型、プロトコル変更。
- **翻訳の改善**：中文 / English / 日本語 ドキュメント。
- **例やツールの追加**：バリデータ、ジェネレーター、リーダー。

## 始め方

1. リポジトリをフォークしてクローンします。
2. ブランチを作成：`git checkout -b fix/descriptive-name`。
3. 変更を加えます。
4. ローカルチェックを実行（[チェック](#チェック) 参照）。
5. 明確なコミットメッセージでコミットし、pull request を開きます。

## 仕様変更のプロセス

この仕様は**規範的**であり、変更はプロデューサーとリーダーの両方に影響します。以下の手順に従ってください：

1. まず **issue** を開き、問題と提案する変更を説明します。
2. 影響を議論：既存パッケージを壊すか？後方互換か？
3. 影響を受けるドキュメントの**全言語版**（中文 / English / 日本語）を更新します。
4. メタデータが影響を受ける場合は `schema/` の対応する **JSON Schema** も更新します。
5. **CHANGELOG** にエントリを追加し、適切にバージョンを上げます：
   - パッチ（`2.0.1`）：明確化、修正、後方互換の追加。
   - マイナー（`2.1.0`）：後方互換の機能追加。
   - メジャー（`3.0.0`）：破壊的変更。

## 翻訳ルール

各言語ディレクトリ（`zh-CN/`、`en/`、`ja/`）は同じ構造を持ちます：

```
<lang>/
├── README.md
├── spec/       # 規範ドキュメント
└── wiki/       # ガイドと用語集
```

- 全言語で**同じ構造とリンク**を維持します。
- **コードブロック、JSON 例、ファイルパス、メッセージ名**は言語非依存であり、**翻訳してはなりません**。
- 規範ドキュメントが変更されたら、全言語ディレクトリを同期します。

## チェック

プッシュ前にローカルで実行：

```bash
# すべての JSON Schema がパース可能か検証
python -c "
import json, glob
for f in glob.glob('schema/*.json'):
    json.load(open(f, encoding='utf-8'))
print('all schemas valid')
"
```

## コミットメッセージスタイル

- 命令形を使用：`Add data-w to char-slot spec` / `Fix broken link in EN spec`。
- スコープ接頭辞：`spec:`、`schema:`、`docs:`、`i18n:`。
- 例：`i18n: translate body-xhtml.md to ja`

## 行動規範

すべての参加者は[行動規範](CODE_OF_CONDUCT.md)に従う必要があります。

## ライセンス

貢献することで、あなたの貢献が [CC0 1.0 Universal（パブリックドメイン献呈）](LICENSE) に基づきリリースされることに同意したものとみなします。
