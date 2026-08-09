# 埋め込みレンダラーとリーダーの IPC 通信仕様 v2.0.1-beta1

> 本仕様は**レンダラー（`renderer/` 内の Web アプリ）とリーダー（ホスト）の通信**を定義します。
> メッセージ形式、キーイベント捕捉、設定配信、リソースリクエストを含みます。

## 1. アーキテクチャと転送

- レンダラーは `renderer/` 内の Web アプリ（`index.html` + `engine.js` + `style.css` + `manifest.json`）。
- リーダーは `index.html` にエンジン/スタイルをインライン化してから、**WebView / iframe** に読み込みます。
- 転送：**`window.postMessage`**。レンダラーは `window.parent.postMessage(msg, '*')` で送信。リーダー側は注入した**ブリッジ層**が捕捉してホストへ転送します。
- **リーダーがホスト、レンダラーは子フレーム**。レンダラーは ZIP/ファイルシステムに直接アクセスせず、すべてのリソースを `renderer:resource` で要求し、リーダーが応答します。

### 1.1 ブリッジ層（リーダーが注入）

リーダーはブリッジスクリプトを注入します：
- `window` の `message` を監視し、`source === 'renderer'` のメッセージをホストへ転送。
- `window.__eipfInit / __eipfOpen / __eipfResource` を公開し、ホストが呼び出して `reader:*` メッセージをレンダラーへ配信。
- `window.__eipfNext / __eipfPrev / __eipfConfig / __eipfState / __eipfDestroy` をキー処理・設定用に公開。

## 2. レンダラー探索（三層）

リーダーは `renderer/manifest.json` を Entry → Album → Series の順に探索します：

| 優先度 | 位置 |
|---|---|
| 1 | Entry ファイル |
| 2 | Album ファイル |
| 3 | Series ファイル |

`manifest.type === "embedded-renderer"` のみ埋め込みレンダラーとみなします。三層すべてで見つからない場合は共有スクリプトによる縮退レンダリングへフォールバックします。

## 3. 読み込みフロー

1. `manifest.entry`（`index.html`）を読み取る。
2. `manifest.engine`（`engine.js`）をインライン化し、`<script src="engine.js">` を置き換える。
3. `manifest.style`（`style.css`）を `</head>` の前にインライン化。
4. ブリッジスクリプトを `</body>` の前に注入。
5. ページを読み込む。
6. ページ読み込み完了後（`onPageFinished`）：`reader:init`（`config.eink` 付き）を送信 → `reader:open`（本文と `startIndex` 付き）を送信。

## 4. 共通メッセージ形式

```typescript
interface EIPFMessage {
  type: string;
  source: 'reader' | 'renderer';
  id?: string;      // リクエスト-レスポンスのマッチング用
  payload?: any;
}
```

受信側は**必ず `source` を検証**し、未知の送信元からのメッセージを無視します。

## 5. リーダー → レンダラー（reader → renderer）

| メッセージ | タイミング | payload |
|---|---|---|
| `reader:init` | ページ準備完了 | `{ css, config: { eink } }` |
| `reader:config` | 実行時の設定変更 | `{ eink }` |
| `reader:open` | 章を開く | `{ entryId, xhtml, metadata, startIndex }` |
| `reader:resource` | リソース応答 | `{ data, mime }` または `{ error, message }`（`id` 付き） |
| `reader:destroy` | 閉じる | `{}` |

### reader:init

```json
{
  "type": "reader:init", "source": "reader",
  "payload": { "css": "", "config": { "eink": true } }
}
```

- `config.eink`：電子ペーパー / 低リフレッシュ最適化スイッチ。`true` でレンダラーはタイプライターとトランジションアニメーションを無効化。

### reader:open

```json
{
  "type": "reader:open", "source": "reader", "id": "open_1",
  "payload": {
    "entryId": "entry_001",
    "xhtml": "<!DOCTYPE html>...body.xhtml 原文...",
    "metadata": {},
    "startIndex": 0
  }
}
```

- `xhtml`：Entry の `body.xhtml` 原文。
- `startIndex`：**再開ターゲット項目インデックス**（レンダラーの `currentIndex`）。`>0` のときはその項目までリプレイし、それ以外は最初から。

### reader:resource（応答）

```json
{ "type": "reader:resource", "source": "reader", "id": "res_0",
  "payload": { "data": "<ArrayBuffer>", "mime": "image/png" } }
```

失敗時：`{ "error": "not_found", "message": "..." }`。

### reader:config

```json
{ "type": "reader:config", "source": "reader", "payload": { "eink": false } }
```

実行時に低リフレッシュ最適化を切り替えます。

## 6. レンダラー → リーダー（renderer → reader）

| メッセージ | タイミング | payload |
|---|---|---|
| `renderer:ready` | 初期化完了 | `{ version, capabilities }` |
| `renderer:rendered` | 章のレンダリング完了 | `{ entryId, dialogueCount, sceneCount, hasChoices }` |
| `renderer:state` | 進行 / 再開ごと | `{ currentIndex, totalEntries, type, progress }` |
| `renderer:resource` | リソース要求 | `{ path }`（`id` 付き） |
| `renderer:ended` | 章の終了 | `{ entryId }` |
| `renderer:choice:selected` | 選択肢が選ばれた | `{ index, value }` |
| `renderer:config:applied` | 設定適用完了 | `{ eink }` |
| `renderer:log` | ログ | `{ level, message }` |
| `renderer:error` | エラー | `{ code, message }` |
| `renderer:navigate` | 遷移要求 | `{ target, ctrlcmd }` |

### renderer:ready

```json
{ "type": "renderer:ready", "source": "renderer",
  "payload": { "version": "2.0.1-beta1", "capabilities": ["scenario","dialogue","typewriter","characters","backgrounds","choices","navigation","export"] } }
```

### renderer:state

```json
{ "type": "renderer:state", "source": "renderer",
  "payload": { "currentIndex": 5, "totalEntries": 120, "type": "dialogue", "progress": 0.0417 } }
```

リーダーは `currentIndex` から**項目単位の正確な進捗**を保存します。

### renderer:resource（要求）

```json
{ "type": "renderer:resource", "source": "renderer",
  "id": "res_0", "payload": { "path": "resource/backgrounds/bg.png" } }
```

リーダーは `path` を三層探索で解決し、`reader:resource` で応答します（`data` はリソースバイト、`mime` は拡張子から推定）。

### renderer:ended

```json
{ "type": "renderer:ended", "source": "renderer", "payload": { "entryId": "entry_001" } }
```

リーダーはこれで次の章へ進みます。

### renderer:navigate

```json
{ "type": "renderer:navigate", "source": "renderer",
  "payload": { "target": "home", "ctrlcmd": "[GotoPage(dest=home)]" } }
```

レンダラーが別ページへの遷移を要求します。ターゲットの意味はリーダーが実装します。

## 7. キーイベント捕捉

キー / タッチイベントは**リーダーのホスト層で捕捉**し、ブリッジ経由でレンダラーを駆動します。レンダラー自身は物理キーを監視しません：

| イベント | リーダーの処理 | 対象 API |
|---|---|---|
| 前のページ | 捕捉 → レンダラーを呼ぶ | `window.__eipfPrev()` |
| 次のページ | 捕捉 → レンダラーを呼ぶ | `window.__eipfNext()` |
| 画面中央タップ | リーダーが捕捉 → ナビバー表示/非表示（ネイティブ） | レンダラーへ転送しない |
| 会話 / 本文タップ | レンダラー内部で進行（自身の click） | レンダラー内で処理 |

### ブリッジ制御 API（レンダラーが公開）

| グローバル関数 | 説明 |
|---|---|
| `window.__eipfNext()` | 次のページ（タイプライター省略 → 進行 → 終端で `renderer:ended`） |
| `window.__eipfPrev()` | 前のページ（前の会話までリプレイ） |
| `window.__eipfConfig(eink)` | 低リフレッシュ最適化の設定（`reader:config` を送信） |
| `window.__eipfState()` | `{ currentIndex, totalEntries }` を返す（ホスト用） |
| `window.__eipfDestroy()` | レンダラーを破棄（`reader:destroy` を送信） |

## 8. リソース リクエスト-レスポンス シーケンス

```
レンダラー                        リーダー
  │ renderer:resource {id,path} │
  ├──────────────────────────────▶ path を解決（Entry→Album→Series、必要なら復号）
  │ reader:resource {id,data,mime}│
  ◀──────────────────────────────┤
  Blob URL を生成してレンダリング
```

## 9. 低リフレッシュ最適化（eink）

- リーダーは `reader:init` で `config.eink` を送信し、実行時は `reader:config` で切り替えられます。
- 有効時、レンダラーはタイプライターテキストを全文表示し、全レイヤーの `transition` を `none` にして画面リフレッシュを減らします。
- 適用後、レンダラーは `renderer:config:applied` を送信します。

## 10. ライフサイクル

```
reader:init ─▶ reader:open(startIndex) ─▶ (進行/キー) ─▶ renderer:ended ─▶ 次の章
                                                └▶ reader:destroy（閉じる）
```
