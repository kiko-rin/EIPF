# リーダー統合ガイド

> EIPF リーダーは以下の能力をカバーする必要があります。リーダーは解凍、レイヤー解析、リソース探索、レンダラー読み込みを担当します。

## 1. ファイル読み込み

1. ZIP ライブラリで選択ファイルを読み込む。
2. `path -> コンテンツ` のマップを構築（ディレクトリはスキップ）。
3. レイヤー判定：`series.json` > `album.json` > `index.json`、それ以外はエラー。
4. タイトル読み取り：`meta.title`（l10n オブジェクト、現在の言語を優先）、フォールバックはファイル名。

## 2. 三層ファイルモデル

リーダーは 3 つのファイルマップを保持します：

| フィールド | レイヤー | 説明 |
|---|---|---|
| Series ファイル | Series | 外側 ZIP の全ファイル |
| Album ファイル | Album | 現在読み込み中の Album のファイル |
| Entry ファイル | Entry | 現在読み込み中の Entry のファイル |

### ネスト ZIP の解凍

- Album 読み込み：Series レイヤーから `album.path` のエントリを取得して解凍。
- Entry 読み込み：まず Album レイヤー、次に Series レイヤーの Entry ZIP を探索。解凍後、全ファイルが共通プレフィックスを共有する場合は自動的に剥離。

## 3. 三層リソース探索

あらゆるリソースは Entry → Album → Series の順で解決します：

```
現在の Entry ZIP → 現在の Album ZIP → Series ZIP → 見つからない
```

ヒット後、Blob / Data URL に変換してレンダリング層へ渡します。

## 4. レンダラー読み込み

1. `renderer/manifest.json` を三層で探索し、`type === "embedded-renderer"` を要求。
2. 見つかれば埋め込みレンダラーモード（`renderer-protocol.md` 参照）。
3. 見つからなければ縮退レンダリングモード（`shared/scenario.js` / `scenario.css` を読み込み）。

## 5. 章のレンダリング

1. 現在の Album と Entry を解凍。
2. `body.xhtml` を探索（`body.xhtml` で終わる任意のファイル）。
3. レンダラー読み込みまたは縮退レンダリング。
4. 埋め込みモード：`reader:open` を送信（本文リソースはローカルアクセス可能な形式に置換済み）。
5. 縮退モード：`<style>scenario.css</style>` + `<body>body.xhtml</body>` + `<script>scenario.js</script>` を組み立て、WebView / iframe へ書き込み。

## 6. 章リストと進行

- リスト：Series レイヤーの `albums[]` を展開。タップで Album を解凍し `album.json.entries[]` を読み取り。
- 進行：`renderer:ended` 後に同じ Album の次の Entry → 次の Album → 終了。

## 7. 通信の要点

- すべてのメッセージは `source` フィールドを持ち、受信側は必ず検証します。
- リソースのリクエスト-レスポンスは `id` でマッチング。
- レンダラーはファイルシステムに直接アクセスせず、`renderer:resource` で要求し、リーダーが応答。
- 進捗：`renderer:state` の `currentIndex` から項目単位の正確な進捗を保存。

## 8. 任意のセキュリティ（2.0.1-beta1）

ファイルを開くとき、ルート JSON に `security` 属性がある場合（`eipf-spec.md` §5 参照）、`enforcement` に応じて検証します：

| enforcement | リーダーの動作 |
|---|---|
| `signature` | 埋め込み公開鍵で `contentHash` を検証。失敗時は拒否 |
| `device` | `security/license.json` を検証（署名 + デバイストークン一致 + 期限切れでない） |
| `encryption` | リソースをレンダリング層に渡す前に復号 |

- `enforcement` をサポートしない場合は「新しいリーダーが必要です」と促し、黙ってダウングレードしないこと。
- デバイストークンはセキュアハードウェア（例：Android Keystore の非エクスポートキー）から導出し、パッケージャーによるバインド用に「設定」で表示すること。

## 9. 主要定数

- 共有 CSS パス：`shared/scenario.css`。
- 共有 JS パス：`shared/scenario.js`。
- 本文ファイル：`resource/text/body.xhtml`（サフィックス `body.xhtml` でマッチ）。
- レンダラーエントリ：`renderer/<manifest.entry>`。
