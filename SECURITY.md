# Security policy

This repository distributes LoopGolem installers and detached signed update manifests. It does **not** contain the application's private source code.

## Reporting a vulnerability

**Please do not publish security vulnerabilities, credentials, signing keys, or sensitive logs in public issues.**

Use GitHub's **Report a vulnerability** / private vulnerability reporting feature if it is enabled for this repository. Otherwise, contact the LoopGolem organization maintainers through a private channel and request a secure reporting method before disclosing details.

Include the affected release tag, platform and a minimal description. Avoid attaching tokens, account identifiers, private signing material, or proprietary mission data.

## Authenticity of downloads

- Use the [official Releases page](https://github.com/LoopGolem/loopgolem-releases/releases).
- Never treat an unsigned GitHub asset, checksum, or manifest as proof of Nancode's identity.
- The in-app updater requires a valid detached signature under an independently pinned signing key, as well as matching package identity and SHA-256.
- See [Verify a release](docs/VERIFY-RELEASE.md).

## Disclosure and release status

A publicly accessible repository is **not** evidence that a production release is approved. Until signed installers and an independently verified production public key are available, there is no authenticated in-app update distribution.
