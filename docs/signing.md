# Code signing

How Unyt's installers are signed, and how to open and wire the accounts behind that, is written up once for both release repos in
[`unyt-sandbox/docs/signing.md`](https://github.com/unytco/unyt-sandbox/blob/main/docs/signing.md). This repo's `release-tauri-app.yaml` and `scripts/check-build-credentials.sh` are identical to that repo's, so the runbook applies here unchanged. Tracked in Linear as UNYT-1093.

What matters when changing signing here:

- **macOS:** the workflow reads the `APPLE_*` secrets, shared with this repo as `unytco` organization secrets. Releases are signed with a `Developer ID Application` certificate and notarized.
- **Windows:** installers are not signed yet, so users meet SmartScreen's "unknown publisher" prompt (see [`windows-clean-machine-check.md`](windows-clean-machine-check.md)).
- **Credential guard:** `scripts/check-build-credentials.sh` fails CI if a build job reads a secret outside its allowlist or holds a write permission such as `id-token: write`. A new signing credential for the build, such as an Azure identity for Windows signing or an App Store Connect API key for notarization, therefore needs the guard's owners to agree how it reaches the build.
- **`signCommand`:** do not commit a Windows `bundle.windows.signCommand` to the shared app config. Inject it in CI through the merged config (`--config ../identity.json`).

Older personal notes, kept for the key-generation steps:

- Tauri keys: https://hackmd.io/@zo-el/S1vu7IFrxx
- Apple certificates: https://hackmd.io/@zo-el/SkARhyw9ke
- Android signing: https://hackmd.io/@zo-el/B1sHELKHxe

## When going to the Apple App Store

The certificate would have to change. We use a `Developer ID Application` certificate today (for notarization), and the App Store needs `Apple Distribution`. The Tauri config would also need to know it is an App Store build, which is not notarized the same way.
