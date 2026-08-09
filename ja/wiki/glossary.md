# 用語集

| 用語 | 説明 |
|---|---|
| **EIPF** | Electric Interactive Publications Format、ZIP ベースの対話型デジタル出版物フォーマット |
| **Series（.eipfs）** | 最外層コンテナ。`series.json`、レンダラー、共有エンジン、複数の Album を含む |
| **Album（.eipfa）** | 中層コンテナ。`album.json` と複数の Entry を含む |
| **Entry（.eipf）** | 最小コンテンツ単位。`index.json`、`main.xml`、マニフェスト、`body.xhtml` を含む |
| **body.xhtml** | Entry のコンテンツ本文。線形の `scene-entry` 列でコンテンツを保持 |
| **scene-entry** | コンテンツ項目型：dialog / Image / music / char / decision / sticker / controller など |
| **char-slot** | `char` 項目内のキャラクタースロット。キャラクター ID、立ち絵パス、キャンバス位置比を含む |
| **bgm-change** | 項目から独立した音楽変化レコード |
| **manifest.xml** | Entry の `META-INF/manifest.xml` ファイルマニフェスト。sha256 チェックサム付き |
| **renderer/manifest.json** | 埋め込みレンダラーマニフェスト。`type` は `embedded-renderer` 固定 |
| **埋め込みレンダラー** | EIPF に同梱される Web アプリ。body.xhtml を対話型ページにレンダリング |
| **三層探索** | リソースを Entry → Album → Series の順に解決する仕組み |
| **IPC / postMessage プロトコル** | リーダーとレンダラー間の通信プロトコル（`renderer-protocol.md` 参照） |
| **ブリッジ層** | リーダーが注入するスクリプト。レンダラーの `postMessage` をホストへ転送し、`__eipfInit/Open/Resource/Next/Prev/Config/State/Destroy` を公開 |
| **startIndex** | `reader:open` の再開ターゲット項目インデックス（レンダラーの `currentIndex`） |
| **低リフレッシュ最適化（eink）** | `reader:init/config` でレンダラーのタイプライターとアニメーションを無効化し、画面リフレッシュを削減 |
| **縮退レンダリングモード** | 埋め込みレンダラーがない場合に `shared/scenario.js` / `.css` で直接レンダリング |
| **sharedResources** | series.json / album.json で共有エンジンとレンダラーを指すパス設定 |
| **l10n オブジェクト** | 多言語フィールド。例：`{ "zh-CN": "..." }` |
| **formatVersion** | 仕様バージョン識別子。現在 `"2.0.1-beta1"`（`"2.0.0"` と互換） |
| **security** | 任意のセキュリティ属性（署名 / デバイスバインド / 暗号化）。`eipf-spec.md` §5 参照 |
