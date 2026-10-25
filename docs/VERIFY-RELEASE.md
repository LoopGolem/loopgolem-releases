# Verifying an official LoopGolem release

**No production release or signing key is published yet.** The steps below apply only after Nancode has approved a release and published an independently authenticated public signing key/fingerprint.

Download the installer and `loopgolem-update.json` from the **assets attached to the same release** on [GitHub Releases](https://github.com/LoopGolem/loopgolem-releases/releases).

## What is verified?

An official in-app update uses a pinned publisher key to authenticate a signed payload and checks the exact installer bytes against it. The payload contains the repository, product, publisher, Windows x64 target, version, filename, size, SHA-256 digest and expiry. All fields must agree with the intended installed application.

**Important:** a checksum and a public key copied from the same untrusted download location do not, by themselves, authenticate the publisher. Obtain the expected public-key fingerprint through a separately trusted publisher channel before accepting a release. Never use a public key supplied solely by an unverified manifest.

## Windows: inspect the installer hash

In PowerShell, from the folder containing your downloaded assets:

```powershell
$version = "<approved-version>"
$installer = "LoopGolem-$version-win-x64-setup.exe"
Get-FileHash -LiteralPath $installer -Algorithm SHA256
```

You can inspect (but **not authenticate**) the signed payload's declared hash:

```powershell
$envelope = Get-Content -Raw -LiteralPath .\loopgolem-update.json | ConvertFrom-Json
$payloadBytes = [Convert]::FromBase64String($envelope.payload)
$payload = [Text.Encoding]::UTF8.GetString($payloadBytes) | ConvertFrom-Json
$payload | Select-Object repository,version,fileName,length,sha256,issuedUtc,expiresUtc
```

**Do not trust this hash yet:** until the detached signature is checked against an independently approved signing public key, an attacker could replace both the installer and its manifest.

## Verify the detached signature

After receiving the approved ECDSA P-256 public key PEM by a trusted channel, an administrator can use PowerShell 7 with an appropriate .NET runtime. Replace the placeholder file with the **independently authenticated** public key; never load an arbitrary key from the release itself.

```powershell
$approvedPublicKeyPem = "C:\trusted\loopgolem-release-public.pem"
$envelope = Get-Content -Raw -LiteralPath .\loopgolem-update.json | ConvertFrom-Json
$payloadBytes = [Convert]::FromBase64String($envelope.payload)
$signature = [Convert]::FromBase64String($envelope.signature)
$key = [Security.Cryptography.ECDsa]::Create()
try {
    $key.ImportFromPem([IO.File]::ReadAllText($approvedPublicKeyPem))
    if ($key.KeySize -ne 256 -or $signature.Length -ne 64) { throw "Unsupported signing key or signature" }
    $valid = $key.VerifyData(
        $payloadBytes, $signature,
        [Security.Cryptography.HashAlgorithmName]::SHA256,
        [Security.Cryptography.DSASignatureFormat]::IeeeP1363FixedFieldConcatenation)
    if (-not $valid) { throw "Invalid signature: do not run this installer" }
} finally { $key.Dispose() }
"Signature verified against the supplied public key."
```

**This snippet verifies only the cryptographic signature.** Before installing, separately verify that the key fingerprint and `keyId` are the publisher-approved ones; the signed `repository` is `LoopGolem/loopgolem-releases`, `publisher` is `Nancode`, product identity and platform are correct, `version` and exact filename match the release, issue/expiry times are valid, and the installer matches both signed `length` and `sha256`.

For instance:

```powershell
$payload = [Text.Encoding]::UTF8.GetString($payloadBytes) | ConvertFrom-Json
if ($payload.repository -cne "LoopGolem/loopgolem-releases" -or
    $payload.publisher -cne "Nancode" -or
    $payload.platform -cne "windows" -or $payload.architecture -cne "x64") {
    throw "Unexpected release identity"
}
$installer = Join-Path (Get-Location) $payload.fileName
if ((Get-Item -LiteralPath $installer).Length -ne [long]$payload.length -or
    (Get-FileHash -LiteralPath $installer -Algorithm SHA256).Hash -ine $payload.sha256) {
    throw "Installer mismatch"
}
```

These snippets do not replace the complete application trust policy. The approved LoopGolem updater performs additional checks and never installs an update without an explicit user action.

## What to do if verification fails

Do not execute the installer. Report the issue privately to the maintainers using an approved security channel. Never post private keys, authentication tokens, or sensitive local diagnostic logs publicly.
