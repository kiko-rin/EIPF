# EIPF — Electric Interactive Publications Format

> **EIPF v2.0.1-beta1** — An open, IP-neutral specification for interactive digital publications.
> 中文 → [README.md](README.md) ／ 日本語 → [README_JA.md](README_JA.md)

![Spec](https://img.shields.io/badge/EIPF-v2.0.1--beta1-4C8CFF)
![License](https://img.shields.io/badge/license-CC0--1.0-blue)
![Docs](https://img.shields.io/badge/docs-zh--CN%20%7C%20en%20%7C%20ja-8A6BBE)

## Introduction

**EIPF (Electric Interactive Publications Format)** is an open, ZIP-based specification for interactive digital publications. It defines:

- **Three-layer nested container structure**: `Series (.eipfs)` → `Album (.eipfa)` → `Entry (.eipf)`, all ZIP containers.
- **A normative content model**: each Entry's `resource/text/body.xhtml` carries content as a linear sequence of `scene-entry` items.
- **An embedded renderer protocol**: postMessage IPC between the renderer (a Web application) and the reader (host).
- **Optional security extension**: `signature` / `device` / `encryption` for copyright protection.

The specification is **generic and IP-neutral**; it does not bind to any specific work, vendor, or intellectual property. Producers map their own content conventions onto the generic structures defined here.

Documentation is maintained in **中文 / English / 日本語** — see the index below.

## Documentation

### Core Specification (spec)

| Document | 中文 | English | 日本語 |
|---|---|---|---|
| Core spec: ZIP container, three-layer architecture, JSON / XML structures, optional security | [`zh-CN/spec/eipf-spec.md`](zh-CN/spec/eipf-spec.md) | [`en/spec/eipf-spec.md`](en/spec/eipf-spec.md) | [`ja/spec/eipf-spec.md`](ja/spec/eipf-spec.md) |
| Content format: body.xhtml structure and scene-entry item types | [`zh-CN/spec/body-xhtml.md`](zh-CN/spec/body-xhtml.md) | [`en/spec/body-xhtml.md`](en/spec/body-xhtml.md) | [`ja/spec/body-xhtml.md`](ja/spec/body-xhtml.md) |
| IPC protocol between embedded renderer and reader | [`zh-CN/spec/renderer-protocol.md`](zh-CN/spec/renderer-protocol.md) | [`en/spec/renderer-protocol.md`](en/spec/renderer-protocol.md) | [`ja/spec/renderer-protocol.md`](ja/spec/renderer-protocol.md) |
| renderer/manifest.json field definitions | [`zh-CN/spec/renderer-manifest.md`](zh-CN/spec/renderer-manifest.md) | [`en/spec/renderer-manifest.md`](en/spec/renderer-manifest.md) | [`ja/spec/renderer-manifest.md`](ja/spec/renderer-manifest.md) |

### Guides & Glossary (wiki)

| Document | 中文 | English | 日本語 |
|---|---|---|---|
| Producer guide: how to generate EIPF files | [`zh-CN/wiki/producer-guide.md`](zh-CN/wiki/producer-guide.md) | [`en/wiki/producer-guide.md`](en/wiki/producer-guide.md) | [`ja/wiki/producer-guide.md`](ja/wiki/producer-guide.md) |
| Reader integration guide: parsing, cascading lookup, renderer loading | [`zh-CN/wiki/reader-integration.md`](zh-CN/wiki/reader-integration.md) | [`en/wiki/reader-integration.md`](en/wiki/reader-integration.md) | [`ja/wiki/reader-integration.md`](ja/wiki/reader-integration.md) |
| Glossary | [`zh-CN/wiki/glossary.md`](zh-CN/wiki/glossary.md) | [`en/wiki/glossary.md`](en/wiki/glossary.md) | [`ja/wiki/glossary.md`](ja/wiki/glossary.md) |

### JSON Schemas (language-neutral)

| Schema | File |
|---|---|
| `series.json` — Series layer | [`schema/series.schema.json`](schema/series.schema.json) |
| `album.json` — Album layer | [`schema/album.schema.json`](schema/album.schema.json) |
| `index.json` — Entry layer | [`schema/entry.schema.json`](schema/entry.schema.json) |
| `renderer/manifest.json` — embedded renderer manifest | [`schema/renderer-manifest.schema.json`](schema/renderer-manifest.schema.json) |

### Language Directories

| Language | Directory |
|---|---|
| 中文 (Chinese) | [`zh-CN/`](zh-CN/README.md) |
| English | [`en/`](en/README.md) |
| 日本語 (Japanese) | [`ja/`](ja/README.md) |
| JSON Schemas | [`schema/`](schema/README.md) |

## Quick Overview

- **Three-layer nesting**: `Series (.eipfs)` → `Album (.eipfa)` → `Entry (.eipf)`, all ZIP containers.
- **Format detection**: determined by the root file `series.json` / `album.json` / `index.json`.
- **Shared resources**: resolved by cascading lookup Entry → Album → Series.
- **Content**: each Entry contains `resource/text/body.xhtml` (XHTML scenario body).
- **Normative content model**: the linear `scene-entry` sequence in body.xhtml is the only normative content; embedded renderers and shared scripts are optional enhancements.
- **IP-neutral**: a generic format, not bound to any specific IP.
- **Optional security**: since 2.0.1-beta1, `signature` / `device` / `encryption` are supported for copyright protection, disabled by default.

## Version

| Item | Value |
|---|---|
| `formatVersion` | `"2.0.1-beta1"` (compatible with `"2.0.0"`) |
| Text encoding | UTF-8 (no BOM) |
| ZIP encryption / split | Disallowed by default (except optional security extension) |
| Namespaces | `urn:eipf:spec:2.0` (main.xml), `urn:eipf:manifest:2.0` (manifest.xml) |

## Reading Order

```
eipf-spec.md → body-xhtml.md → renderer-manifest.md → renderer-protocol.md
→ producer-guide.md → reader-integration.md
```

## Contributing

- [Contributing Guide](CONTRIBUTING.md) ([中文](CONTRIBUTING.zh-CN.md) / [日本語](CONTRIBUTING.ja.md))
- [Code of Conduct](CODE_OF_CONDUCT.md) · [Security Policy](SECURITY.md) · [Changelog](CHANGELOG.md)
- [Citation](CITATION.cff) · [License](LICENSE) (CC0 1.0 Public Domain)
