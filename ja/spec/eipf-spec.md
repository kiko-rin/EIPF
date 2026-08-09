# EIPF コア仕様 v2.0.1-beta1

> EIPF（Electric Interactive Publications Format）は、ZIP ベースの対話型デジタル出版物フォーマットです。
> 本仕様は**汎用・IP 中立**であり、各プロデューサは自作品の規約をここで定義される汎用構造にマッピングします。

## 1. 全体アーキテクチャ

EIPF は三層の入れ子コンテナ構造を持ちます：

```
Series (.eipfs)
├── series.json
├── renderer/
├── shared/
│   ├── scenario.css
│   └── scenario.js
├── resource/
│   └── render/manifest.json
└── albums/
    └── *.eipfa
        ├── album.json
        └── entries/
            └── *.eipf
                ├── index.json
                ├── main.xml
                ├── META-INF/manifest.xml
                ├── resource/text/body.xhtml
                └── resource/{backgrounds|characters|illustrations|audio|video}/...
```

### 1.0 規範的コンテンツモデル

- **EIPF は汎用の対話型デジタル出版物フォーマット**であり、特定の作品・IP に依存しません。作品固有の命名（アクティビティシリーズ、章番号、キャラクター ID など）はプロデューサが定義するもので、本仕様の範囲外です。
- コンテンツは Entry の `body.xhtml` に保持され、その内容は**線形の `scene-entry` 項目列**です（`body-xhtml.md` を参照）。
- リーダーは `scene-entry` 項目のパースを規範的実装とすべきです。埋め込みレンダラー（`renderer/`）は**任意の**拡張であり、リーダーはこれに依存してコンテンツを表示してはなりません。

### 1.1 レイヤー判定

ルートファイルを以下の順に読み、レイヤーを判定します：

| 優先度 | ルートファイル | レイヤー |
|---|---|---|
| 1 | `series.json` | Series |
| 2 | `album.json` | Album |
| 3 | `index.json` | Entry |

いずれも一致しない場合、EIPF ファイルではないと判定します。

### 1.2 拡張子と MIME

| レイヤー | 拡張子 | MIME |
|---|---|---|
| Entry | `.eipf` | `application/eipf+zip` |
| Album | `.eipfa` | `application/eipf+zip` |
| Series | `.eipfs` | `application/eipf+zip` |

## 2. ZIP コンテナ

- 圧縮アルゴリズム：Deflate（`ZIP_DEFLATED`）または Store（`ZIP_STORED`）。
- 推奨：外側の `.eipfs` はテキスト系（`.json .xml .xhtml .css .js .html .txt .svg`）に Deflate、バイナリリソースに Store；内側の `.eipf` / `.eipfa` はすべて Store。
- ファイル名は UTF-8。
- 暗号化と分割アーカイブはデフォルト禁止（任意のセキュリティ拡張を除く、§5 参照）。
- 入れ子をサポート（Entry ZIP は Album ZIP 内、Album ZIP は Series ZIP 内）。

## 3. レイヤー構造

### 3.1 Entry レイヤー（.eipf）

#### ディレクトリ構造

```
<id>.eipf
├── index.json                  # 必須、Entry メタデータ
├── main.xml                    # 必須、構造スケルトン
├── META-INF/
│   └── manifest.xml            # 必須、ファイルマニフェスト
└── resource/
    ├── text/body.xhtml         # 必須、コンテンツ本文
    ├── backgrounds/            # 背景画像（resource/backgrounds/...）
    ├── characters/             # キャラクター立ち絵（resource/characters/...）
    ├── illustrations/          # イラスト（resource/illustrations/...）
    ├── audio/                  # 音声（resource/audio/...）
    └── video/                  # CG ビデオ（resource/video/...）
```

#### index.json

```json
{
  "id": "entry_001",
  "formatVersion": "2.0.1-beta1",
  "contentType": "scenario",
  "language": ["zh-CN"],
  "title": { "zh-CN": "第1章 開幕" },
  "version": "1.0.0",
  "structure": {
    "root": "main.xml",
    "text": "resource/text/body.xhtml"
  },
  "resources": {
    "backgrounds": "resource/backgrounds/",
    "characters": "resource/characters/",
    "illustrations": "resource/illustrations/",
    "audio": "resource/audio/",
    "video": "resource/video/"
  },
  "renderer": {
    "enabled": true,
    "manifest": "renderer/manifest.json"
  }
}
```

フィールド一覧：

| フィールド | 型 | 説明 |
|---|---|---|
| `id` | string | Entry の一意識別子 |
| `formatVersion` | string | `"2.0.1-beta1"`（`"2.0.0"` と互換） |
| `contentType` | string | `"scenario"` |
| `language` | string[] | BCP 47 言語リスト |
| `title` | object | 多言語タイトル `{ "zh-CN": ... }` |
| `version` | string | コンテンツバージョン |
| `structure.root` | string | 構造スケルトンファイル `"main.xml"` |
| `structure.text` | string | 本文ファイル `"resource/text/body.xhtml"` |
| `resources.*` | string | 各リソースカテゴリのディレクトリパス |
| `renderer.enabled` | boolean | 埋め込みレンダラーを有効にするか |
| `renderer.manifest` | string | レンダラーマニフェストパス `"renderer/manifest.json"` |
| `security` | object | **任意**、セキュリティ属性（§5 参照） |

#### main.xml

名前空間：`urn:eipf:spec:2.0`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<document xmlns="urn:eipf:spec:2.0">
  <head>
    <meta charset="UTF-8"/>
    <title>第1章 開幕</title>
    <language>zh-CN</language>
  </head>
  <body>
    <text src="resource/text/body.xhtml"/>
  </body>
</document>
```

#### META-INF/manifest.xml

名前空間：`urn:eipf:manifest:2.0`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest xmlns="urn:eipf:manifest:2.0"
          version="2.0.1-beta1"
          generatedAt="2026-08-09T00:00:00Z"
          generator="EIPF Publisher">
  <file path="index.json" mediaType="application/json" checksum="sha256:..." size="123"/>
  <file path="main.xml" mediaType="application/xml" checksum="sha256:..." size="456"/>
  <file path="resource/text/body.xhtml" mediaType="application/xhtml+xml" checksum="sha256:..." size="789"/>
</manifest>
```

- 各 `<file>` は ZIP 内の 1 ファイルを宣言します。
- `checksum` の形式：`sha256:<hex>`。

### 3.2 Album レイヤー（.eipfa）

#### album.json

```json
{
  "id": "act_01",
  "formatVersion": "2.0.1-beta1",
  "contentType": "scenario",
  "language": ["zh-CN"],
  "title": { "zh-CN": "第1巻" },
  "version": "1.0.0",
  "entries": [
    {
      "id": "entry_001",
      "path": "entries/entry_001.eipf",
      "sortOrder": 1,
      "title": { "zh-CN": "第1章 開幕" },
      "description": { "zh-CN": "全42セリフ" }
    }
  ],
  "sharedResources": {
    "scenarioCss": "shared/scenario.css",
    "scenarioJs": "shared/scenario.js",
    "renderer": "renderer/manifest.json"
  }
}
```

`entries[]` フィールド：

| フィールド | 型 | 説明 |
|---|---|---|
| `id` | string | Entry ID |
| `path` | string | Entry ファイルパス `"entries/<id>.eipf"` |
| `sortOrder` | number | ソート順（1 から増加） |
| `title` | object | 多言語タイトル |
| `description` | object | 概要 |

### 3.3 Series レイヤー（.eipfs）

#### series.json

```json
{
  "id": "sample-series",
  "formatVersion": "2.0.1-beta1",
  "contentType": "scenario",
  "language": ["zh-CN"],
  "title": { "zh-CN": "サンプルコレクション" },
  "description": { "zh-CN": "サンプルコンテンツ集" },
  "creators": [
    { "name": "Publisher A", "role": "developer", "url": "https://example.com" }
  ],
  "publisher": "Publisher A",
  "publishedAt": "2026-08-09",
  "version": "1.0.0",
  "albums": [
    {
      "id": "act_01",
      "path": "albums/act_01.eipfa",
      "sortOrder": 1,
      "title": { "zh-CN": "第1巻" }
    }
  ],
  "sharedResources": {
    "scenarioCss": "shared/scenario.css",
    "scenarioJs": "shared/scenario.js",
    "renderer": "renderer/manifest.json"
  }
}
```

`albums[]` フィールド：

| フィールド | 型 | 説明 |
|---|---|---|
| `id` | string | Album ID |
| `path` | string | Album ファイルパス `"albums/<id>.eipfa"` |
| `sortOrder` | number | ソート順 |
| `title` | object | 多言語タイトル |

## 4. レンダー出力マニフェスト

Series レイヤーは `resource/render/manifest.json`（レンダー出力メタデータ）を含むことができます：

```json
{
  "formatVersion": "2.0.1-beta1",
  "generator": "EIPF Publisher",
  "renderedAt": "2026-08-09T00:00:00Z",
  "sourceXhtml": "resource/text/body.xhtml",
  "resultFile": "result.html",
  "scenes": [
    { "sceneId": "global", "dialogueCount": 0, "title": "サンプル" }
  ]
}
```

## 5. 任意のセキュリティ属性（2.0.1-beta1 で追加）

`security` は**任意**の属性であり、プロデューサがコンテンツを保護する必要がある場合にのみ `series.json` / `album.json` / `index.json` に書き込みます。未使用のパッケージは 2.0.0 と完全に互換です。

### 5.1 構造

```json
"security": {
  "formatVersion": "2.0.1-beta1",
  "enforcement": "signature+device+encryption",
  "signatureAlgorithm": "RSASSA-PKCS1-v1_5-SHA256",
  "hashAlgorithm": "SHA-256",
  "contentHash": "sha256:<hex>",
  "signature": "<base64 署名>",
  "encryption": {
    "cipher": "AES-256-GCM",
    "keyWrapAlgorithm": "RSA-PKCS1v15-SHA256",
    "wrappedKey": "<base64>",
    "encryptedPrefixes": ["resource/", "renderer/"],
    "packageId": "<パッケージID>"
  }
}
```

| フィールド | 説明 |
|---|---|
| `enforcement` | 保護レベル。組み合わせ可能：`signature` / `signature+device` / `signature+encryption` / `signature+device+encryption` |
| `contentHash` | パッケージ内の全コンテンツファイルの正規化 SHA-256（メタデータファイルと `security/` を除く） |
| `signature` | `contentHash` へのプロデューサ秘密鍵による PKCS#1 v1.5（SHA-256）署名 |
| `encryption.cipher` | リソース暗号化アルゴリズム `"AES-256-GCM"` |
| `encryption.wrappedKey` | リーダーの RSA 公開鍵でラップしたコンテンツキー（base64） |
| `encryption.encryptedPrefixes` | 暗号化されるパスプレフィックス |

### 5.2 デバイスバインディング（device）

パッケージには `security/license.json` が含まれる必要があります（対象デバイスが判明した時点でプロデューサが注入）：

```json
{
  "formatVersion": "2.0.1-beta1",
  "packageId": "<パッケージID>",
  "deviceToken": "<デバイストークン>",
  "issuedAt": "2026-08-09T12:00:00Z",
  "expiresAt": "",
  "signature": "<base64>"
}
```

- `deviceToken`：リーダーのセキュアハードウェア（例：Android Keystore）が生成する非エクスポートなアイデンティティから導出され、「このパッケージはこのデバイスだけにライセンスされる」ことを証明します。
- `signature`：ライセンス本文（`signature` フィールドを除く）へのプロデューサ秘密鍵署名。

### 5.3 リーダーの動作

| `enforcement` | リーダーが必須で検証すること |
|---|---|
| `signature` | 埋め込み公開鍵で `contentHash` を検証；失敗時は拒否 |
| `device` | `security/license.json` を読み、署名検証 + `deviceToken` がローカルと一致 + 期限切れでないこと |
| `encryption` | `encryptedPrefixes` に一致するパスをレンダリング層に渡す前に復号 |

- `enforcement` を**サポートしない**リーダーは「新しいリーダーが必要です」と促し、黙ってダウングレードしてはなりません。
- `signature` のみでは暗号化を伴いません。`device` / `encryption` はコピーやリバースエンジニアリングの障壁を高めます。クライアント側検証は「コストを上げる」手段であり、プロによるリバースエンジニアリングを絶対に防げるわけではありません。

### 5.4 推奨レベル

1. デフォルトは `security` なし（2.0.0 と完全互換）。
2. 改ざん・偽造対策：`signature`。
3. 共有防止・デバイスバインド：`signature+device`。
4. 完全なコンテンツ保護：`signature+device+encryption`。

## 6. リソースパス規則

- すべてのリソースパス（body.xhtml の `data-url`、`src`、`href` など）は**EIPF ルートからの相対パス**。
- リソースは `resource/{rtype}/{filename}` に格納。`rtype` ∈ `backgrounds` / `characters` / `illustrations` / `audio` / `video`。
- 分類の推奨：
  - ビデオ：`.mp4` / `.webm`、または項目型 `video`。
  - 音声：`.mp3` / `.ogg`、または項目型 `music`。
  - 背景：項目型 `Image` で背景の意味を持つもの。
  - キャラクター：項目型 `char` の立ち絵。
  - その他：`illustrations`。
