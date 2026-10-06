# cloakbrowser-pro-binaries

Public **GitHub Releases** mirror for [CloakHQ/cloakbrowser](https://github.com/CloakHQ/cloakbrowser) **Pro** Chromium builds, keyed by version and platform.

Used by [finger-chromium / stealthbrowser](https://github.com/ootonn/stealthbrowser) `license-gateway` sync (`GITHUB_MIRROR_REPO`) so clients can download via `STEALTHBROWSER_LICENSE_ORIGIN` (302 or local cache) without hitting Cloak API on every machine.

## Release layout

| Item | Convention |
|------|------------|
| Git tag | `chromium-v{version}-pro` (example: `chromium-v152.0.7977.82.1-pro`) |
| Windows asset | `cloakbrowser-windows-x64.zip` |
| Linux x64 asset | `cloakbrowser-linux-x64.tar.gz` |
| Linux arm64 asset | `cloakbrowser-linux-arm64.tar.gz` |

Version strings and archive names match Cloak Pro releases and signed `SHA256SUMS` on the upstream tag when sync verification succeeds.

## How assets get here

On a trusted machine with a valid Pro `license.key`:

```powershell
# In finger-chromium repo
$env:GITHUB_MIRROR_REPO = "ootonn/cloakbrowser-pro-binaries"
$env:GITHUB_TOKEN = "<token with repo scope>"
.\scripts\sync-pro-mirror.ps1
```

Sync downloads each platform once from Cloak (or GitHub when available), verifies integrity where possible, writes under `~/.cloakbrowser/gateway-mirror`, and uploads release assets to this repository.

## Disclaimer

Binaries are **Pro** builds; use only if your license allows. This mirror is maintained for private infrastructure convenience and is **not** affiliated with CloakHQ or StealthHQ. Do not commit `license.key` or tokens to any repository.
