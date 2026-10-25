# LoopGolem — Official Releases

This is the **public, binary-only distribution repository** for **LoopGolem**, published by **Nancode**.

> **Status:** Distribution infrastructure is being prepared. No installer is approved for public release yet.

## Downloads

Approved installers will appear in [GitHub Releases](https://github.com/LoopGolem/loopgolem-releases/releases) when available. Only download files from the release assets of this repository.

A Windows x64 release is expected to contain:

| Release asset | Purpose |
| --- | --- |
| `LoopGolem-<version>-win-x64-setup.exe` | Windows installer |
| `loopgolem-update.json` | Detached, cryptographically signed update manifest |
| Optional checksums / release notes | Human inspection and release information |

Release tags use `v<version>` (for example, `v0.1.0-preview.9`). Versions shown here are illustrative, **not published releases**.

## Update security

LoopGolem is designed to check this repository for updates without credentials. **Downloading and installing an update requires explicit user consent.**

For in-app installation, the application validates an ECDSA P-256 signature using a trusted publisher key embedded in the installed application, then verifies the exact installer version, product, platform, architecture, size and SHA-256 digest. A manifest or checksum placed in this repository is **not** a substitute for a verified publisher signature.

If a release lacks a valid trusted signature, the application must refuse in-app installation.

Read [how to verify a release](docs/VERIFY-RELEASE.md) or the [release publishing policy](docs/RELEASE-POLICY.md).

## Scope and licensing

This repository hosts **distribution metadata and release binaries only**. It is not the LoopGolem source-code repository. No application source, internal CI logs, credentials, private signing keys or development artifacts belong here.

**Public repository visibility does not make the application open source or grant permission to redistribute or modify it.** Any applicable application license will be provided separately by the publisher.

## Security reports

Please do not publish potential vulnerabilities, credentials or signing material in public issues. Contact the maintainers privately via a supported GitHub security reporting channel, when available.
