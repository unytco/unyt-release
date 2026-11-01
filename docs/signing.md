# Signing required during setup

## Generating keys for Tauri

https://hackmd.io/@zo-el/S1vu7IFrxx

## Create Apple Certificates for Code Signing

https://hackmd.io/@zo-el/SkARhyw9ke

## Android Signing Setup for Tauri

https://hackmd.io/@zo-el/B1sHELKHxe

## Windows: Azure Artifact Signing

The Windows binaries and installers are signed with Azure Artifact Signing (formerly Trusted Signing), with an Individual identity validation. No certificate file exists: the key lives in Azure, and the certificate is short-lived and reissued by the service. The build job on each Windows row of `release-tauri-app.yaml` installs `artifact-signing-cli` and writes a `bundle.windows.signCommand` into the app's `tauri.conf.json`, so Tauri signs the main binary, each sidecar, the NSIS installer and the MSI with it before the updater signs them.

What the release needs:

- In `release-tauri-app.yaml`, the step "Sign every Windows binary and installer with Azure Artifact Signing" names the signing **endpoint** (such as `https://eus.codesigning.azure.net`), the **account** and the **certificate profile**. These are public, and a reviewer sees them in the diff.
- Three repository secrets, the service principal that may ask the service to sign: `AZURE_TENANT_ID`, `AZURE_CLIENT_ID` and `AZURE_CLIENT_SECRET`. They come from an app registration in Microsoft Entra ID that holds the role **Trusted Signing Certificate Profile Signer** on the signing account, and nothing more. The client secret expires (24 months at most): signing fails on that day, and the smoke's "Authenticode-signed and trusted" check turns red.
- `scripts/check-build-credentials.sh` lists these three among the build secrets, since a credential that can only sign during the build is in the same class as the Apple ones.

The smoke declares the installers signed (`$script:ExpectWindowsSigned` in `scripts/smoke/check-windows.ps1`), so an unsigned installer fails a release. A newly issued certificate still draws SmartScreen's reputation warning until enough machines have run files signed by it; see `windows-clean-machine-check.md`.

A later step is a federated credential (OIDC) on the same app registration, so no `AZURE_CLIENT_SECRET` exists at all. That needs `id-token: write` on the build job, an `azure/login` step, and a carve-out for `id-token` in the guard's read-only rule.

## When Going to the Apple App Store

**Important**
We would have to change the certificate we are using.

- Currently we use a `Developer ID Application` (For notrization)
- But when we go to the app store we need to use `Apple Distribution`
- This may need some changes to the tauri-config to know its for the appstore so it wont need to notarize
