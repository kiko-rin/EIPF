# 制作ガイド：EIPF ファイルの生成

> 本ガイドでは、コンテンツソースを EIPF ファイルにする方法を説明します。コンテンツソースの具体的な形式はプロデューサが決定します。本ガイドは EIPF 側の構造制約のみを規定します。

## 1. 前提

- コンテンツソース：各作品のスクリプト / アセット（テキスト、画像、音声、ビデオ）。
- パッケージングツールまたはスクリプト：コンテンツソースを解析 → `body.xhtml` を生成 → 三層 ZIP を組み立て。
- リソースマッピング：コンテンツソース内のリソース識別子をダウンロード可能な URL に解決。

## 2. パッケージングフロー（推奨 5 フェーズ）

| フェーズ | 説明 |
|---|---|
| Phase 1 | 各シーンを解析 → body.xhtml を生成 → リソース一覧を抽出 → リソースパスマップを構築 |
| Phase 2 | 全リソースをダウンロード（失敗時リトライ ≤3、URL 小文字化フォールバック） |
| Phase 3 | 各 Entry ZIP を構築 → Album ZIP を組み立て → Series album メタデータを収集 |
| Phase 4 | `renderer/` と `shared/`（scenario.css / scenario.js）をコピー |
| Phase 5 | `series.json` とレンダーマニフェストを書き、最終 `.eipfs` をパッケージ |

## 3. 生成規則チートシート

### Entry ZIP の内容（推奨 `ZIP_STORED`）

- `index.json`
- `main.xml`
- `META-INF/manifest.xml`
- `resource/text/body.xhtml`
- `resource/{rtype}/{filename}`（参照されたリソース）

### Album ZIP の内容（`ZIP_STORED`）

- `album.json`
- `entries/<id>.eipf`（Entry ごとにネスト ZIP ひとつ）

### Series ZIP（外側。テキストは Deflate / バイナリは Store）

- `series.json`
- `renderer/`
- `shared/scenario.css`、`shared/scenario.js`
- `resource/render/manifest.json`（任意）
- `albums/<id>.eipfa`

## 4. ID 生成

- Entry / Album ID：ソースパスまたはタイトルをサニタイズ（`<>:"/\|?*` と空白を除去し `_` に置換）。一意かつ合法的なファイル名を保証。
- アルバムタイトル：プロデューサがコンテンツカテゴリに応じて生成。

## 5. リソース分類とダウンロード

分類：`audio` / `backgrounds` / `characters` / `illustrations` / `video`。

ダウンロード方針：

- ターゲットパス `resource/{rtype}/{filename}`。
- ファイルが既にあればスキップ。
- 失敗時リトライ：5xx、タイムアウト、接続エラーのみ。最大 3 回。
- URL 小文字化フォールバック：元のリンクが失敗した後に小文字化して再試行。

## 6. 検証

全ファイルについて `sha256:` プレフィックスのチェックサムを計算し、`META-INF/manifest.xml` に書き込んで整合性を検証できるようにします。

## 7. 任意のセキュリティ（2.0.1-beta1）

著作権保護のため、パッケージ時に `security` 属性を有効化できます（`eipf-spec.md` §5 参照）：

- **signature**：プロデューサの秘密鍵でコンテンツハッシュに署名。改ざん・偽造対策。
- **device**：対象デバイストークンにバインドした `security/license.json` を注入。コピー・共有防止。
- **encryption**：`resource/` と `renderer/` を AES-256-GCM で暗号化。解凍して内容を見ることを防止。

実装の提案：

- 鍵ペア（RSA-2048）：プロデューサが秘密鍵を保持し、公開鍵をリーダーに埋め込み。
- デバイストークン：リーダーのセキュアハードウェアから導出。ユーザーが「設定」でコピーし、パッケージャに貼り付け。
- 注意：クライアント側の検証は「コストを上げる」手段であり、プロによるリバースエンジニアリングを絶対に防ぐものではありません。
