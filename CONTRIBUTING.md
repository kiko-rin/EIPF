# Contributing to EIPF

Thank you for your interest in contributing to **EIPF (Electric Interactive Publications Format)**.

This repository is a **documentation and specification project** — it contains normative
Markdown documents and JSON Schemas. Please read this guide before opening issues or
pull requests.

> 中文贡献指南见 [CONTRIBUTING.zh-CN.md](CONTRIBUTING.zh-CN.md)
> 日本語の貢献ガイドは [CONTRIBUTING.ja.md](CONTRIBUTING.ja.md)

## Ways to Contribute

- **Report a bug** in a document or schema (incorrect example, broken link, ambiguity).
- **Propose a specification change** (new field, new `scene-entry` type, protocol change).
- **Improve translations** (中文 / English / 日本語 documentation).
- **Add examples or tooling** (validators, generators, readers).

## Getting Started

1. Fork the repository and clone it.
2. Create a branch: `git checkout -b fix/descriptive-name`.
3. Make your changes.
4. Run the local checks (see [Checks](#checks)).
5. Commit with a clear message and open a pull request.

## Specification Change Process

The specification is **normative** — changes affect producers and readers. Follow this process:

1. Open an **issue** first describing the problem and the proposed change.
2. Discuss the impact: does it break existing packages? Is it backward compatible?
3. Update **all language versions** of the affected document (中文 / English / 日本語).
4. Update the corresponding **JSON Schema** under `schema/` if metadata is affected.
5. Add a **CHANGELOG** entry and bump the version appropriately:
   - Patch (`2.0.1`): clarification, fixes, non-breaking additions.
   - Minor (`2.1.0`): backward-compatible feature additions.
   - Major (`3.0.0`): breaking changes.

## Translation Rules

Each language folder (`zh-CN/`, `en/`, `ja/`) mirrors the same structure:

```
<lang>/
├── README.md
├── spec/       # normative documents
└── wiki/       # guides and glossary
```

- Keep the **same structure and links** across languages.
- **Code blocks, JSON examples, file paths, and message names** are language-neutral
  and MUST NOT be translated.
- Keep every language folder in sync when a normative document changes.

## Checks

Run these locally before pushing:

```bash
# Validate all JSON Schemas are parseable
python -c "
import json, glob
for f in glob.glob('schema/*.json'):
    json.load(open(f, encoding='utf-8'))
print('all schemas valid')
"
```

## Commit Message Style

- Use imperative mood: `Add data-w to char-slot spec` / `Fix broken link in EN spec`.
- Prefix for scoped changes: `spec:`, `schema:`, `docs:`, `i18n:`.
- Example: `i18n: translate body-xhtml.md to ja`

## Code of Conduct

All participants must follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## License

By contributing, you agree that your contributions are released under the
[CC0 1.0 Universal (Public Domain Dedication)](LICENSE).
