# EIPF Core Specification v2.0.1-beta1

> EIPF (Electric Interactive Publications Format) is a ZIP-based format for interactive digital publications.
> This specification is **generic and IP-neutral**; producers map their own content conventions onto the generic structures defined here.

## 1. Overall Architecture

EIPF uses a three-layer nested container structure:

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

### 1.0 Normative Content Model

- **EIPF is a generic format for interactive digital publications**, independent of any specific work or IP. Work-specific naming (activity series, chapter numbers, character IDs, etc.) is defined by producers and is out of scope for this specification.
- Content is carried by the Entry's `body.xhtml`, whose content is a **linear sequence of `scene-entry` items** (see `body-xhtml.md`).
- Readers SHOULD implement parsing of `scene-entry` items as the normative behavior; the embedded renderer (`renderer/`) is an **optional** enhancement and readers MUST NOT depend on it to display content.

### 1.1 Layer Detection

The layer is determined by reading the root file in the following order:

| Priority | Root file | Layer |
|---|---|---|
| 1 | `series.json` | Series |
| 2 | `album.json` | Album |
| 3 | `index.json` | Entry |

If none is recognized, the file is not a valid EIPF file.

### 1.2 Extensions and MIME Types

| Layer | Extension | MIME |
|---|---|---|
| Entry | `.eipf` | `application/eipf+zip` |
| Album | `.eipfa` | `application/eipf+zip` |
| Series | `.eipfs` | `application/eipf+zip` |

## 2. ZIP Container

- Compression: Deflate (`ZIP_DEFLATED`) or Store (`ZIP_STORED`).
- Recommendation: the outer `.eipfs` uses Deflate for text files (`.json .xml .xhtml .css .js .html .txt .svg`) and Store for binary resources; inner `.eipf` / `.eipfa` use Store throughout.
- File names are UTF-8 encoded.
- Encryption and split archives are disallowed by default (except the optional security extension, see §5).
- Nesting is supported (Entry ZIP inside Album ZIP, Album ZIP inside Series ZIP).

## 3. Layer Structure

### 3.1 Entry Layer (.eipf)

#### Directory Structure

```
<id>.eipf
├── index.json                  # required, Entry metadata
├── main.xml                    # required, structure skeleton
├── META-INF/
│   └── manifest.xml            # required, file manifest
└── resource/
    ├── text/body.xhtml         # required, content body
    ├── backgrounds/            # background images (resource/backgrounds/...)
    ├── characters/             # character sprites (resource/characters/...)
    ├── illustrations/          # illustrations (resource/illustrations/...)
    ├── audio/                  # audio (resource/audio/...)
    └── video/                  # CG video (resource/video/...)
```

#### index.json

```json
{
  "id": "entry_001",
  "formatVersion": "2.0.1-beta1",
  "contentType": "scenario",
  "language": ["zh-CN"],
  "title": { "zh-CN": "Chapter 1 Opening" },
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

Field reference:

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique Entry identifier |
| `formatVersion` | string | `"2.0.1-beta1"` (compatible with `"2.0.0"`) |
| `contentType` | string | `"scenario"` |
| `language` | string[] | BCP 47 language list |
| `title` | object | Localized title `{ "zh-CN": ... }` |
| `version` | string | Content version |
| `structure.root` | string | Structure skeleton file, `"main.xml"` |
| `structure.text` | string | Content body file, `"resource/text/body.xhtml"` |
| `resources.*` | string | Directory path for each resource category |
| `renderer.enabled` | boolean | Whether the embedded renderer is enabled |
| `renderer.manifest` | string | Renderer manifest path, `"renderer/manifest.json"` |
| `security` | object | **Optional**, security attributes (see §5) |

#### main.xml

Namespace: `urn:eipf:spec:2.0`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<document xmlns="urn:eipf:spec:2.0">
  <head>
    <meta charset="UTF-8"/>
    <title>Chapter 1 Opening</title>
    <language>zh-CN</language>
  </head>
  <body>
    <text src="resource/text/body.xhtml"/>
  </body>
</document>
```

#### META-INF/manifest.xml

Namespace: `urn:eipf:manifest:2.0`.

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

- Each `<file>` declares one file inside the ZIP.
- `checksum` format: `sha256:<hex>`.

### 3.2 Album Layer (.eipfa)

#### album.json

```json
{
  "id": "act_01",
  "formatVersion": "2.0.1-beta1",
  "contentType": "scenario",
  "language": ["zh-CN"],
  "title": { "zh-CN": "Volume 1" },
  "version": "1.0.0",
  "entries": [
    {
      "id": "entry_001",
      "path": "entries/entry_001.eipf",
      "sortOrder": 1,
      "title": { "zh-CN": "Chapter 1 Opening" },
      "description": { "zh-CN": "42 dialogues in total" }
    }
  ],
  "sharedResources": {
    "scenarioCss": "shared/scenario.css",
    "scenarioJs": "shared/scenario.js",
    "renderer": "renderer/manifest.json"
  }
}
```

`entries[]` fields:

| Field | Type | Description |
|---|---|---|
| `id` | string | Entry ID |
| `path` | string | Entry file path, `"entries/<id>.eipf"` |
| `sortOrder` | number | Sort order (increments from 1) |
| `title` | object | Localized title |
| `description` | object | Brief description |

### 3.3 Series Layer (.eipfs)

#### series.json

```json
{
  "id": "sample-series",
  "formatVersion": "2.0.1-beta1",
  "contentType": "scenario",
  "language": ["zh-CN"],
  "title": { "zh-CN": "Sample Collection" },
  "description": { "zh-CN": "Sample content collection" },
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
      "title": { "zh-CN": "Volume 1" }
    }
  ],
  "sharedResources": {
    "scenarioCss": "shared/scenario.css",
    "scenarioJs": "shared/scenario.js",
    "renderer": "renderer/manifest.json"
  }
}
```

`albums[]` fields:

| Field | Type | Description |
|---|---|---|
| `id` | string | Album ID |
| `path` | string | Album file path, `"albums/<id>.eipfa"` |
| `sortOrder` | number | Sort order |
| `title` | object | Localized title |

## 4. Render Output Manifest

The Series layer MAY contain `resource/render/manifest.json` (render output metadata):

```json
{
  "formatVersion": "2.0.1-beta1",
  "generator": "EIPF Publisher",
  "renderedAt": "2026-08-09T00:00:00Z",
  "sourceXhtml": "resource/text/body.xhtml",
  "resultFile": "result.html",
  "scenes": [
    { "sceneId": "global", "dialogueCount": 0, "title": "Sample" }
  ]
}
```

## 5. Optional Security Attributes (added in 2.0.1-beta1)

`security` is an **optional** attribute written to `series.json` / `album.json` / `index.json` only when the producer needs to protect content. Packages without it are fully compatible with 2.0.0.

### 5.1 Structure

```json
"security": {
  "formatVersion": "2.0.1-beta1",
  "enforcement": "signature+device+encryption",
  "signatureAlgorithm": "RSASSA-PKCS1-v1_5-SHA256",
  "hashAlgorithm": "SHA-256",
  "contentHash": "sha256:<hex>",
  "signature": "<base64 signature>",
  "encryption": {
    "cipher": "AES-256-GCM",
    "keyWrapAlgorithm": "RSA-PKCS1v15-SHA256",
    "wrappedKey": "<base64>",
    "encryptedPrefixes": ["resource/", "renderer/"],
    "packageId": "<package id>"
  }
}
```

| Field | Description |
|---|---|
| `enforcement` | Protection level, combinable: `signature` / `signature+device` / `signature+encryption` / `signature+device+encryption` |
| `contentHash` | Canonical SHA-256 of all content files in the package (excluding metadata files and `security/`) |
| `signature` | Producer's private-key signature of `contentHash` (PKCS#1 v1.5 / SHA-256) |
| `encryption.cipher` | Resource encryption algorithm, `"AES-256-GCM"` |
| `encryption.wrappedKey` | Content key wrapped with the reader's RSA public key, base64 |
| `encryption.encryptedPrefixes` | Encrypted path prefixes |

### 5.2 Device Binding (device)

The package MUST contain `security/license.json` (injected by the producer once the target device is known):

```json
{
  "formatVersion": "2.0.1-beta1",
  "packageId": "<package id>",
  "deviceToken": "<device token>",
  "issuedAt": "2026-08-09T12:00:00Z",
  "expiresAt": "",
  "signature": "<base64>"
}
```

- `deviceToken`: derived from a non-exportable identity generated by the reader's secure hardware (e.g., Android Keystore); used to prove "this package is licensed only to this device".
- `signature`: the producer's private-key signature of the license body (excluding the `signature` field).

### 5.3 Reader Behavior

| `enforcement` | Reader MUST verify |
|---|---|
| `signature` | Verify `contentHash` with the embedded public key; reject on failure |
| `device` | Read `security/license.json`: verify signature + `deviceToken` matches the local token + not expired |
| `encryption` | Decrypt paths matching `encryptedPrefixes` before handing them to the rendering layer |

- A reader that does **not** support `enforcement` MUST prompt "a newer reader is required" instead of silently downgrading.
- `signature` alone does not involve encryption; `device`/`encryption` raise the bar for copying and reverse engineering. Client-side verification is a "cost-raising" measure and cannot absolutely defeat professional reverse engineering.

### 5.4 Recommended Levels

1. No `security` by default (fully compatible with 2.0.0).
2. Tamper/forgery resistance: `signature`.
3. Copy-prevention / device binding: `signature+device`.
4. Full content protection: `signature+device+encryption`.

## 6. Resource Path Rules

- All resource paths (in body.xhtml `data-url`, `src`, `href`, etc.) are **relative to the EIPF root**.
- Resources are stored at `resource/{rtype}/{filename}` where `rtype` ∈ `backgrounds` / `characters` / `illustrations` / `audio` / `video`.
- Classification suggestion:
  - Video: `.mp4` / `.webm`, or item type `video`.
  - Audio: `.mp3` / `.ogg`, or item type `music`.
  - Background: item type `Image` with background semantics.
  - Character: sprites of item type `char`.
  - Everything else: `illustrations`.
