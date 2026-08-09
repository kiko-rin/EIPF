# renderer/manifest.json フィールド仕様 v2.0.1-beta1

> 埋め込みレンダラーのマニフェスト。リーダーはこれでレンダラーを識別・読み込みます。

## 1. 例

```json
{
  "name": "Default EIPF Scenario Renderer",
  "type": "embedded-renderer",
  "version": "1.0.0",
  "entry": "index.html",
  "engine": "engine.js",
  "style": "style.css",
  "capabilities": ["scenario", "dialogue", "typewriter", "characters", "backgrounds", "choices", "navigation", "export"]
}
```

## 2. フィールド一覧

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `name` | string | ★ | レンダラー名 |
| `type` | string | ★ | 固定 `"embedded-renderer"`（リーダーがこれで識別） |
| `version` | string | ★ | レンダラーバージョン |
| `entry` | string | ★ | エントリ HTML ファイル名（例：`index.html`） |
| `engine` | string | | エンジン JS ファイル名（例：`engine.js`）。リーダーが entry へインライン化 |
| `style` | string | | スタイルファイル名（例：`style.css`）。リーダーが entry へインライン化 |
| `capabilities` | string[] | | レンダラーの能力リスト |

## 3. リーダーの利用方法

- `entry`：エントリ HTML を読み込む（WebView / iframe）。
- `engine`：`<script src="engine">` をインラインスクリプトに置き換え。正確な一致がなければ外部 script を除去し `</body>` の前にインライン化。
- `style`：`</head>` の前に `<style>` をインライン化。
- `capabilities`：レンダラーが `renderer:ready` で報告。リーダーは能力表示に利用できる。
- **IPC**：読み込み後、リーダーはブリッジ層を注入し、`renderer-protocol.md` に従って双方向通信（`reader:init/open/config`、`renderer:state/ended` など。キーめくり `__eipfNext/__eipfPrev` と低リフレッシュ `eink` 設定を含む）。

## 4. 互換性

- `formatVersion`、`communication`、`output`、`sandbox` は歴史的な草案の概念であり、**必須ではありません**。リーダーは未知のフィールドを無視すべきです。
