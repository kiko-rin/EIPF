# Security Policy

## Supported Versions

| Version | Supported |
|---|---|
| 2.0.1-beta1 | ✅ |
| 2.0.0 | ✅ (compatibility) |
| < 2.0.0 | ❌ |

## Reporting a Vulnerability

EIPF is a **specification and documentation project**. Security-relevant issues fall into two categories:

### 1. Security extension (`security` attribute)

Since 2.0.1-beta1, EIPF supports optional `signature` / `device` / `encryption`
attributes for copyright protection (see `zh-CN/spec/eipf-spec.md` §5).

Please report:

- **Cryptographic weaknesses** in the specified algorithms or their usage
  (signature scheme, key wrapping, AES-GCM nonce handling, device-token derivation).
- **Bypass vectors** in the documented reader verification flow
  (e.g., downgrade attacks, license replay, decryption key extraction).

### 2. Content parsing

- Malformed ZIP structures that could cause a reader to crash or loop.
- Path traversal via resource paths (`../`, absolute paths).

## How to Report

Do **not** open a public issue for confirmed or suspected security
vulnerabilities. Instead, report privately via the project's GitHub Security
Advisories ("Report a vulnerability") or the repository owner's contact email.

Please include:

- Affected version and document/schema.
- A description of the issue and its impact.
- Reproduction steps (preferably a minimal malformed package or script).
- Suggested fix, if any.

## Response Expectations

- Acknowledgment within **7 days**.
- Status update and mitigation plan within **30 days**.
- Coordinated disclosure: we will credit reporters unless they request otherwise.

## Honest Scope Note

The client-side security extension raises the cost of copying and reverse
engineering; it is **not** absolute protection against professional
reverse engineering on rooted devices. This is documented in the specification.
