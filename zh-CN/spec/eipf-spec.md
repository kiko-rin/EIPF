# EIPF 核心规范 v2.0.1-beta1

> EIPF（Electric Interactive Publications Format）是基于 ZIP 的交互式数字出版物格式。
> 本规范为**通用、作品无关**的内容格式标准；制作者可将自有内容约定映射到本规范定义的通用结构上。

## 1. 总体架构

EIPF 采用三层嵌套容器结构：

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

### 1.0 规范内容模型

- **EIPF 是通用交互式数字出版物格式**，与任何具体作品/IP 无关。各作品的专属命名（活动系列、章节编号、角色 ID 等）由制作者自行约定，不属于本规范。
- 内容承载以 Entry 的 `body.xhtml` 为准，其内容是**线性 `scene-entry` 条目序列**（见 `body-xhtml.md`）。
- 阅读器以解析 `scene-entry` 为规范实现；内嵌渲染器（`renderer/`）是**可选**的补充能力，阅读器不应依赖它才能显示内容。

### 1.1 层级判定

读取文件时按以下顺序判定层级：

| 优先级 | 根文件 | 层级 |
|---|---|---|
| 1 | `series.json` | Series |
| 2 | `album.json` | Album |
| 3 | `index.json` | Entry |

无法识别时判定为非 EIPF 文件。

### 1.2 扩展名与 MIME

| 层级 | 扩展名 | MIME |
|---|---|---|
| Entry | `.eipf` | `application/eipf+zip` |
| Album | `.eipfa` | `application/eipf+zip` |
| Series | `.eipfs` | `application/eipf+zip` |

## 2. ZIP 容器

- 压缩算法：Deflate（`ZIP_DEFLATED`）或 Store（`ZIP_STORED`）。
- 建议：外层 `.eipfs` 对文本类（`.json .xml .xhtml .css .js .html .txt .svg`）用 Deflate，二进制资源用 Store；内层 `.eipf` / `.eipfa` 全部使用 Store。
- 文件名编码：UTF-8。
- 默认禁止加密、分卷（可选安全扩展除外，见 §5）。
- 支持嵌套（Entry ZIP 内嵌于 Album ZIP，Album ZIP 内嵌于 Series ZIP）。

## 3. 层级结构

### 3.1 Entry 层（.eipf）

#### 目录结构

```
<id>.eipf
├── index.json                  # 必填，Entry 元数据
├── main.xml                    # 必填，结构骨架
├── META-INF/
│   └── manifest.xml            # 必填，文件清单
└── resource/
    ├── text/body.xhtml         # 必填，内容正文
    ├── backgrounds/            # 背景图（resource/backgrounds/...）
    ├── characters/             # 角色立绘（resource/characters/...）
    ├── illustrations/          # 插图（resource/illustrations/...）
    ├── audio/                  # 音频（resource/audio/...）
    └── video/                  # 视频 CG（resource/video/...）
```

#### index.json

```json
{
  "id": "entry_001",
  "formatVersion": "2.0.1-beta1",
  "contentType": "scenario",
  "language": ["zh-CN"],
  "title": { "zh-CN": "第一章 开场" },
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

字段说明：

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | string | Entry 唯一标识 |
| `formatVersion` | string | `"2.0.1-beta1"`（兼容 `"2.0.0"`） |
| `contentType` | string | `"scenario"` |
| `language` | string[] | BCP 47 语言列表 |
| `title` | object | 多语言标题 `{ "zh-CN": ... }` |
| `version` | string | 内容版本 |
| `structure.root` | string | 结构骨架文件，`"main.xml"` |
| `structure.text` | string | 正文文件，`"resource/text/body.xhtml"` |
| `resources.*` | string | 各资源类别目录路径 |
| `renderer.enabled` | boolean | 是否启用内嵌渲染器 |
| `renderer.manifest` | string | 渲染器清单路径，`"renderer/manifest.json"` |
| `security` | object | **可选**，安全属性（见 §5） |

#### main.xml

命名空间：`urn:eipf:spec:2.0`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<document xmlns="urn:eipf:spec:2.0">
  <head>
    <meta charset="UTF-8"/>
    <title>第一章 开场</title>
    <language>zh-CN</language>
  </head>
  <body>
    <text src="resource/text/body.xhtml"/>
  </body>
</document>
```

#### META-INF/manifest.xml

命名空间：`urn:eipf:manifest:2.0`。

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

- 每个 `<file>` 声明 ZIP 内一个文件。
- `checksum` 格式：`sha256:<hex>`。

### 3.2 Album 层（.eipfa）

#### album.json

```json
{
  "id": "act_01",
  "formatVersion": "2.0.1-beta1",
  "contentType": "scenario",
  "language": ["zh-CN"],
  "title": { "zh-CN": "第一卷" },
  "version": "1.0.0",
  "entries": [
    {
      "id": "entry_001",
      "path": "entries/entry_001.eipf",
      "sortOrder": 1,
      "title": { "zh-CN": "第一章 开场" },
      "description": { "zh-CN": "共 42 条对话" }
    }
  ],
  "sharedResources": {
    "scenarioCss": "shared/scenario.css",
    "scenarioJs": "shared/scenario.js",
    "renderer": "renderer/manifest.json"
  }
}
```

`entries[]` 字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | string | Entry ID |
| `path` | string | Entry 文件路径，`"entries/<id>.eipf"` |
| `sortOrder` | number | 排序（从 1 递增） |
| `title` | object | 多语言标题 |
| `description` | object | 简介 |

### 3.3 Series 层（.eipfs）

#### series.json

```json
{
  "id": "sample-series",
  "formatVersion": "2.0.1-beta1",
  "contentType": "scenario",
  "language": ["zh-CN"],
  "title": { "zh-CN": "示例合集" },
  "description": { "zh-CN": "示例内容合集" },
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
      "title": { "zh-CN": "第一卷" }
    }
  ],
  "sharedResources": {
    "scenarioCss": "shared/scenario.css",
    "scenarioJs": "shared/scenario.js",
    "renderer": "renderer/manifest.json"
  }
}
```

`albums[]` 字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | string | Album ID |
| `path` | string | Album 文件路径，`"albums/<id>.eipfa"` |
| `sortOrder` | number | 排序 |
| `title` | object | 多语言标题 |

## 4. 渲染输出清单

Series 层可包含 `resource/render/manifest.json`（渲染输出元数据）：

```json
{
  "formatVersion": "2.0.1-beta1",
  "generator": "EIPF Publisher",
  "renderedAt": "2026-08-09T00:00:00Z",
  "sourceXhtml": "resource/text/body.xhtml",
  "resultFile": "result.html",
  "scenes": [
    { "sceneId": "global", "dialogueCount": 0, "title": "示例" }
  ]
}
```

## 5. 可选安全属性（2.0.1-beta1 新增）

`security` 是**可选属性**，仅在制作者需要保护内容时写入 `series.json` / `album.json` / `index.json`。未启用的包与 2.0.0 完全一致。

### 5.1 属性结构

```json
"security": {
  "formatVersion": "2.0.1-beta1",
  "enforcement": "signature+device+encryption",
  "signatureAlgorithm": "RSASSA-PKCS1-v1_5-SHA256",
  "hashAlgorithm": "SHA-256",
  "contentHash": "sha256:<hex>",
  "signature": "<base64 签名>",
  "encryption": {
    "cipher": "AES-256-GCM",
    "keyWrapAlgorithm": "RSA-PKCS1v15-SHA256",
    "wrappedKey": "<base64>",
    "encryptedPrefixes": ["resource/", "renderer/"],
    "packageId": "<包ID>"
  }
}
```

| 字段 | 说明 |
|---|---|
| `enforcement` | 保护级别，可组合：`signature` / `signature+device` / `signature+encryption` / `signature+device+encryption` |
| `contentHash` | 对包内全部内容文件（除元数据文件与 `security/` 之外）的规范化 SHA-256 |
| `signature` | 制作者私钥对 `contentHash` 的 PKCS#1 v1.5(SHA-256) 签名 |
| `encryption.cipher` | 资源加密算法 `"AES-256-GCM"` |
| `encryption.wrappedKey` | 内容密钥用阅读器 RSA 公钥包装后的 base64 |
| `encryption.encryptedPrefixes` | 被加密路径前缀 |

### 5.2 设备绑定（device）

包内需含 `security/license.json`（制作者在已知目标设备后注入）：

```json
{
  "formatVersion": "2.0.1-beta1",
  "packageId": "<包ID>",
  "deviceToken": "<设备令牌>",
  "issuedAt": "2026-08-09T12:00:00Z",
  "expiresAt": "",
  "signature": "<base64>"
}
```

- `deviceToken`：阅读器侧由安全硬件（如 Android Keystore）生成的不可导出身份派生，用于证明"这一份包只授权给这一台设备"。
- `signature`：制作者私钥对 license 正文（不含 signature 字段）的签名。

### 5.3 阅读器行为约定

| `enforcement` | 阅读器必须做的检查 |
|---|---|
| `signature` | 内置公钥验签 `contentHash`；失败 → 拒绝打开 |
| `device` | 读 `security/license.json`：验签 + `deviceToken` 与本地令牌一致 + 未过期 |
| `encryption` | 对 `encryptedPrefixes` 命中路径先解密再交付渲染层 |

- **不支持** `enforcement` 的阅读器打开受保护包时应提示"需要更高版本阅读器"，而非降级放行。
- 仅 `signature` 时不涉及加密；`device`/`encryption` 用于提高复制与逆向门槛。客户端侧校验属"提高成本"，无法绝对阻止专业逆向。

### 5.4 推荐等级

1. 默认不加 `security`（与 2.0.0 完全兼容）。
2. 防篡改/防伪：`signature`。
3. 防分享/绑定设备：`signature+device`。
4. 防解包看内容：`signature+device+encryption`。

## 6. 资源路径规则

- 所有资源路径（body.xhtml 中的 `data-url`、`src`、`href` 等）**相对于 EIPF 根目录**。
- 资源实际存储于 `resource/{rtype}/{filename}`，其中 `rtype` ∈ `backgrounds` / `characters` / `illustrations` / `audio` / `video`。
- 分类建议：
  - 视频：`.mp4` / `.webm`，或条目类型为 `video`。
  - 音频：`.mp3` / `.ogg`，或条目类型为 `music`。
  - 背景：条目类型为 `Image` 且语义为背景。
  - 角色：条目类型为 `char` 的立绘。
  - 其余归入 `illustrations`。
