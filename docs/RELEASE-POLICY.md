# Release publishing policy

This public repository is a **distribution endpoint**, not a build system or source mirror. Releases are created only after approval by the LoopGolem/Nancode maintainer. Publication of a public release is a separate decision from merging application source code.

## Required asset contract

For each approved release version `<version>`:

- Git tag: `v<version>`
- Windows x64 installer: `LoopGolem-<version>-win-x64-setup.exe`
- Detached signed update envelope: `loopgolem-update.json`
- Optional: `SHA256SUMS.txt` (human-readable checksums) and approved release notes

The updater derives the installer and manifest download URLs from the release tag. Both required assets must be attached to the **same GitHub Release** in `LoopGolem/loopgolem-releases`; keeping them only in the Git tree is not sufficient.

The JSON envelope contains `keyId`, base64-encoded `payload` (the exact UTF-8 bytes signed) and `signature`. The signed payload binds the release to its canonical version, publisher (`Nancode`), LoopGolem product identity, repository (`LoopGolem/loopgolem-releases`), Windows x64 platform, exact filename, byte count, SHA-256, and issue/expiry dates. The signature uses **ECDSA P-256 / SHA-256, IEEE P1363 (64 bytes)**. A release page or an unverified checksum alone is not proof of publisher authenticity.

## Checklist before publishing

1. **Approval and licensing:** explicitly approve public distribution and the applicable application terms. Public visibility does not grant an open-source license.
2. **Source provenance:** build from a reviewed, clean commit in the private development repository. Run all required unit, packaged, installer and disposable two-version upgrade tests. Record the source revision privately.
3. **Signing root:** independently verify the production public-key fingerprint and `keyId`, and embed the scoped public key into the Windows build. Protect the private signing key outside source checkouts, this repository, GitHub artifacts, CI logs and ordinary chat.
4. **Build identity:** configure the binary-only repository as `LoopGolem/loopgolem-releases`; ensure the build embeds that discovery source, trusted public key, and the correct version. Do not overwrite earlier build outputs.
5. **Executable integrity:** validate installer publisher/product/version, the packaged Desktop and Worker versions, all declared dependencies, and the generated provenance/hashes.
6. **Optional Authenticode:** if an approved Windows signing identity is available, Authenticode-sign the final installer **before** computing detached-manifest hashes. Authenticode is recommended in addition to, not instead of, the updater signature.
7. **Detached signing:** sign the exact final installer with the protected ECDSA P-256 production key; the resulting `loopgolem-update.json` should expire within the configured short validity window (default 14 days; maximum 31 days). Never ship synthetic/test signing keys as production roots.
8. **Independent verification:** using an independently authenticated public key, validate the envelope signature, signed identity and exact installer SHA-256/length. Confirm the GitHub release version/tag and asset names agree.
9. **Publication authorization:** only now create/publish the GitHub Release, uploading the two required assets and optional checksums/notes. Use GitHub's `prerelease` flag for preview versions and confirm channel behavior. Do not upload source archives from the private repository, raw build directories, internal logs, credentials, private keys, or unapproved licensing documents.
10. **Post-publication check:** fetch the assets anonymously from their final GitHub URLs, independently recheck integrity/signature, and validate discovery from a deliberately configured test build. Never allow an automated publication path to overwrite an earlier installer under the same version/tag.

## Signing and key management

- The feed cannot redefine, rotate, or revoke the publisher trust root; it is embedded in each compatible application build.
- Keep signing keys out of this Git history, GitHub Actions secrets/artifacts and publicly accessible machines unless a separate, reviewed key-management policy explicitly permits a signing service.
- If a private key is compromised, **stop publication** and plan a new trusted bootstrap. Replacing the JSON envelope alone cannot make compromised roots safe.
- The public repository contains no release-signing automation or credentials by default. Do not enable unattended uploads or signing without a separate reviewed design.

## Versioning and recovery

- Use canonical SemVer without a leading `v` in package metadata; the Git tag adds `v`.
- A preview version is a prerelease. Stable-channel clients exclude prereleases; Preview clients can consider newer prerelease/stable versions.
- Never substitute different installer bytes under an existing version. A renewed signed manifest may reference the **same immutable installer bytes**.
- Inno Setup updates are not atomic rollbacks. Back up state and qualify bootstrap/upgrade/recovery in a disposable Windows VM or GitHub-hosted runner before introducing real users.
- Running, paused, and otherwise recoverable missions must not be interrupted simply to install an update. User consent is required to install.

**Current status:** publication infrastructure and the production signing root are not yet provisioned. This document is a policy, **not** an indication that any installer is currently verified or available.
