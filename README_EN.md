# EIPF — Electric Interactive Publications Format

> **EIPF v2.0.1-beta1** — An open, IP-neutral specification for interactive digital publications.
> 中文 → [README.md](README.md) ／ 日本語 → [README_JA.md](README_JA.md)

![Spec](https://img.shields.io/badge/EIPF-v2.0.1--beta1-4C8CFF)
![License](https://img.shields.io/badge/license-CC0--1.0-blue)
![Docs](https://img.shields.io/badge/docs-zh--CN%20%7C%20en%20%7C%20ja-8A6BBE)

## Documentation (by language)

| Language | Directory |
|---|---|
| 中文 (Chinese) | [`zh-CN/`](zh-CN/README.md) |
| English | [`en/`](en/README.md) |
| 日本語 (Japanese) | [`ja/`](ja/README.md) |
| JSON Schemas (language-neutral) | [`schema/`](schema/README.md) |

## Version

| Item | Value |
|---|---|
| `formatVersion` | `2.0.1-beta1` |
| Text encoding | UTF-8 (no BOM) |
| ZIP encryption / split | Disallowed by default (except optional security extension) |
| Namespaces | `urn:eipf:spec:2.0` (main.xml), `urn:eipf:manifest:2.0` (manifest.xml) |

## Quick Overview

- **Three-layer nesting**: `Series (.eipfs)` → `Album (.eipfa)` → `Entry (.eipf)`, all ZIP containers.
- **Format detection**: determined by the root file `series.json` / `album.json` / `index.json`.
- **Shared resources**: resolved by cascading lookup Entry → Album → Series.
- **Content**: each Entry contains `resource/text/body.xhtml` (XHTML scenario body).
- **Normative content model**: the linear `scene-entry` sequence in body.xhtml is the only normative content; embedded renderers and shared scripts are optional enhancements.
- **IP-neutral**: a generic format, not bound to any specific IP.
- **Optional security**: since 2.0.1-beta1, `signature` / `device` / `encryption` are supported for copyright protection, disabled by default.

## Contributing

- [Contributing Guide](CONTRIBUTING.md) ([中文](CONTRIBUTING.zh-CN.md) / [日本語](CONTRIBUTING.ja.md))
- [Code of Conduct](CODE_OF_CONDUCT.md) · [Security Policy](SECURITY.md) · [Changelog](CHANGELOG.md)
- [License](LICENSE) (CC0 1.0 Public Domain)
