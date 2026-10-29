# Unyt

![GitHub release (latest by date)](https://img.shields.io/github/v/release/unytco/unyt-release?style=for-the-badge)
![GitHub All Releases](https://img.shields.io/github/downloads/unytco/unyt-release/total?style=for-the-badge)

## Intro

Built on [Holochain](https://holochain.org/), Unyt is peer-to-peer accounting infrastructure. With it you create currencies, automate economic agreements, bridge value across networks, and trade directly between any units. No central server holds its records, and it needs no miners and no platform middlemen.

Every participant runs the app on their own device and keeps their own signed chain of records. A full arc node also validates its peers directly. There is no blockchain consensus, no mining, and no third party that holds your funds. The community sets the fees, and it can set them to zero.

This repository releases the Unyt app on MainNet. [unyt-sandbox](https://github.com/unytco/unyt-sandbox) releases the TestNet app, Unyt Sandbox.

## Downloads

Take your installer from the [latest release](https://github.com/unytco/unyt-release/releases/latest) of this repo. The [Releases page](https://github.com/unytco/unyt-release/releases) has every build, including the older ones. A release run publishes its release as a pre-release. A person then makes it the latest release by hand.

Each release ships the same app in two variants. You can run either.

1. The **default-arc** version is a full arc node. It holds a full copy of the DHT locally, and it synchronizes all published data.
2. The **zero-arc** version is lighter to run. It holds only your own history and caches some other network data. Some actions are slower, because it fetches data from peers. It gossips and validates like any other node, but it stores no shard of the DHT.

Take default-arc unless you want the lighter one.

### Reading an installer's name

Every installer is named for what it is, so you can pick yours from the filename alone:

```
unyt_<version>_<product>_<variant>_<architecture>_<platform><extension>
```

| Part | Values |
| --- | --- |
| `<product>` | The app's name, `productName` in [`identity.json`](identity.json), with each space as a dot |
| `<variant>` | `default-arc`, `zero-arc` |
| `<architecture>` | `aarch64` for Apple Silicon, `x64` for 64-bit Intel or AMD on macOS and Windows, `amd64` for the same on Linux |
| `<platform>` | `darwin` for macOS, `linux`, `windows` |
| `<extension>` | macOS `.dmg`, Linux `.deb` or `.AppImage`, Windows `.exe` (setup) or `.msi` |

A release also carries assets that are not installers. Installing the app needs none of them:

- the `.app.tar.gz` bundles
- `unyt.happ`, `unyt.webhapp` and `alliance.dna`, the Holochain application the installers are built around
- the `.sig` signatures, and the `updater-default-arc.json` and `updater-zero-arc.json` manifests that the app's Update reads
- `SHA256SUMS`, the sha256 of every installer, of the Holochain files and of `unyt_cli`
- `SHA256SUMS.minisig`, the signature of `SHA256SUMS` by the key the app's updates are signed with

### Checking a download

Put the download in one folder with `SHA256SUMS` and `SHA256SUMS.minisig`. Then run [minisign](https://jedisct1.github.io/minisign/) and `sha256sum` in that folder:

```sh
minisign -VHm SHA256SUMS -P RWSRDr/xqHIO10FBhhER/XbuGj/nLx5ulUdnLCl1Wdr1+ZYxr52+Orgl
sha256sum --check --ignore-missing SHA256SUMS
```

The key after `-P` is this repo's update key: the second line of `plugins.updater.pubkey` in [`identity.json`](identity.json), base64 decoded. minisign prints a trusted comment that names the release's version. On macOS, use `shasum -a 256 --check --ignore-missing SHA256SUMS`.

A `-dev` release is signed with a throwaway key, so the `minisign` step works for stable releases only.

### On Linux, swapping between the two variants

Both variants install as one package at the same version. Its name is the app's `productName` in lower case, with dashes for spaces. So with either variant installed, `apt install ./unyt_..._linux.deb` finds that version present, changes nothing and exits 0. A graphical software centre uses the same package manager, so it does nothing either. To swap, install the .deb directly:

```sh
sudo dpkg -i ./unyt_*_zero-arc_amd64_linux.deb
```

or make apt reinstall it:

```sh
sudo apt install --reinstall ./unyt_*_zero-arc_amd64_linux.deb
```

The AppImage has none of this. Each file runs on its own.

## unyt_cli

`unyt_cli` has releases of its own, tagged `cli-vX.Y.0.N`. Each is build `N` of the CLI for the app release `vX.Y.0`, whose DNA every `vX.Y.Z` carries.

- **Which one:** for an app release `vX.Y.Z`, take `unyt_cli` from the newest `cli-vX.Y.0.N` release. `unyt_cli --version` prints `X.Y.0.N`. The release's notes are the section `## [X.Y.0.N]` of the pinned app's `crates/unyt_cli/CHANGELOG.md`. A CLI release is a pre-release that carries `unyt_cli` alone, so it is never the repo's latest release. Compare it with the sha256 digest that GitHub shows for it on that release.
- **Build 0:** a `vX.Y.0` release run publishes its release, then makes `cli-vX.Y.0.0` from its own commit. Before you tag `vX.Y.0`, cut the Unreleased entries of the app's `crates/unyt_cli/CHANGELOG.md` into `## [X.Y.0.0]`. A `vX.Y.0-dev.M` run makes `cli-vX.Y.0-dev.M.0`.
- **Releasing newer CLI code:** in unyt-app, cut the Unreleased entries of `crates/unyt_cli/CHANGELOG.md` into a section `## [X.Y.0.N]`, with the next `N`. Here, make a branch from the `vX.Y.0` tag. On it, pin `unyt` to an app commit with that code and that section, which must be on the app's remote. Push the tag `cli-vX.Y.0.N` at that commit.
- **The DNA rule:** the run builds the pinned app's DNA. It publishes nothing unless the `vX.Y.0` release carries that `alliance.dna` byte for byte. A change to the code of `unyt_cli` alone never changes the DNA, so it never blocks a release. A change to what the zomes build from does: `dnas/`, `crates/rave_engine`, their dependencies and the toolchain. If that changed since `vX.Y.0`, the run refuses. In that case, cherry-pick the CLI change onto the app commit that `vX.Y.0` pins.

## Setup

When you open the app for the first time, it makes a pair of public and private keys for you. A private keystore (Lair) on your own machine holds them. In Unyt, your public key is "your address". Others use it to send to you, receive from you, or give you a role.

To reset completely and start again with a new account:

1. Uninstall the app.
2. Delete its local data: on macOS, `~/Library/Application Support/<identifier>`, where `<identifier>` is `identifier` in [`identity.json`](identity.json). On Windows, follow [`docs/windows-wipe.md`](docs/windows-wipe.md).
3. Install the app again. You get a new key pair and a fresh identity.

## Network

[`network.json`](network.json) names the network that a release builds the app for, and the values its build reads. A value in `network.json` or `identity.json` that starts with `TO BE SET` is not set yet, and a release stops before it builds. `bash scripts/release-probe.sh <app-dir>` lists each one.

Each X.Y.0 release and each pre-release runs its own joining, bootstrap, relay and auth services. A patch release uses the services of its X.Y.0 release. So a release puts a version, with dots as dashes, in front of the subdomain of `UNYT_JOINING_SERVICE_URL`, `UNYT_BOOTSTRAP_URL`, `UNYT_RELAY_URL` and `UNYT_AUTH_SERVER_URL`. At a stable tag vX.Y.Z that version is X.Y.0, and at a pre-release it is the whole version:

| Tag | `https://relay.unyt.co` builds as |
| --- | --- |
| `v1.0.0`, `v1.0.1` | `https://1-0-0-relay.unyt.co` |
| `v1.1.0-dev.0` | `https://1-1-0-dev-0-relay.unyt.co` |

If `network.json` sets `UNYT_BOOTSTRAP_URL`, `UNYT_RELAY_URL` or `UNYT_AUTH_SERVER_URL` to `""`, the app uses its own default for that service, and no version is added.

`VITE_MIGRATION_SERVICE_URL`, `VITE_HOT_BRIDGE_URL` and `VITE_VOUCHER_SERVICE_URL` outlive any one app version, so they build as written. The notes of a release list every value as it was built.

## Related Resources

- [Unyt Documentation](https://unyt.co/docs/)
- [Unyt Blog](https://unyt.co/blog/)
- [Smart Agreement Code Library](https://github.com/unytco/smart_agreement_library)
- [Holochain DEV.HC Discord](https://discord.com/invite/k55DS5dmPH): the Unyt channel is under Projects
- [Feedback / Issues](https://github.com/orgs/unytco/projects/5/views/1)

## License

This project is licensed under the terms in the [LICENSE](LICENSE) file.

Copyright (C) 2024 - 2026, unyt.co
