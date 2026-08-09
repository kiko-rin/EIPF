# Changelog

All notable changes to the EIPF specification are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.1-beta1] - 2026-08-09

### Added

- Optional `security` attribute (signature / device binding / encryption) on
  `series.json`, `album.json`, and `index.json`.
  - `signature`: RSASSA-PKCS1-v1_5 (SHA-256) content-hash signing.
  - `device`: `security/license.json` device-token binding.
  - `encryption`: AES-256-GCM resource encryption with RSA-wrapped content key.
- Documented reader enforcement behavior and recommended protection levels
  (see `zh-CN/spec/eipf-spec.md` §5).
- `security` property added to all JSON Schemas under `schema/`.
- `data-w` attribute on `char-slot` (optional canvas-width ratio).

### Changed

- `formatVersion` in produced packages is now `"2.0.1-beta1"`; `"2.0.0"` remains valid.
- Documentation reorganized by language: `zh-CN/`, `en/`, `ja/`.

### Fixed

- `upgrade()` delay conversion guard now recognizes `"2.0.1-beta1"` files.

## [2.0.0] - 2026-06-30

### Added

- Three-layer container structure: Series (.eipfs) → Album (.eipfa) → Entry (.eipf).
- Normative `scene-entry` content model in `body.xhtml`.
- Embedded renderer manifest and IPC protocol definitions.
- JSON Schemas for series / album / entry / renderer-manifest.

