# Passwall LuCI feed

[简体中文](README.md) | **English**

This private source feed is used by the customized
[hnt510-great/openwrt](https://github.com/hnt510-great/openwrt) main tree.
The project targets x86_64/glibc, APK, native systemd and GNOME. Package
selection follows the main tree's saved configuration.

| Item | Value |
| --- | --- |
| Feed name | `passwall_luci` |
| Repository branch | `main` |
| Original upstream | [https://github.com/Openwrt-Passwall/openwrt-passwall](https://github.com/Openwrt-Passwall/openwrt-passwall) (`main`) |

This feed supplies `luci-app-passwall` pages, service scripts, configuration
and translations, together with the proxy cores and DNS tools in
`passwall_packages`. Its integration history is maintained alongside the
private main tree's dependencies and versioning rules.

## Main-tree integration

- Passwall remains the default proxy; daed starts disabled. Their transparent
  interception must not be enabled together. See the main tree's daed guide
  for switching procedures.
- LuCI, UCI, `/etc/init.d/passwall` and the procd service interface remain.
  Native process supervision uses the main tree's procd/systemd integration.
- The main tree uses LuCI Git revision versioning for Passwall code and
  translations independently. Committed changes receive new versions even
  when upstream does not manually increment a release field.
- Uncommitted runtime-file changes cannot rely on Git revision versioning for
  a new APK identity; the package version must still advance before publication.

Use the two Passwall feeds already configured by the main tree. There is no
need to delete recipes from `packages` or clone an extra copy into `package/`.

## Get the source and build

Authenticate on the build machine with an account that can read every private
repository, then obtain the complete main tree:

```sh
gh auth login --hostname github.com --git-protocol https --web
gh auth setup-git
git clone https://github.com/hnt510-great/openwrt.git
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
```

The main tree's `feeds.conf.default` already contains:

```text
src-git passwall_luci https://github.com/hnt510-great/openwrt-passwall-luci.git;main
```

To update only this feed in an existing main tree:

```sh
./scripts/feeds update passwall_luci
./scripts/feeds install -a -p passwall_luci
```

Select the desired package in the main configuration and run `make defconfig`.
Example individual package build:

```sh
make package/feeds/passwall_luci/luci-app-passwall/compile V=s
```

Run these commands from the main-tree root. Preserve an existing `.config`;
see [local build](https://github.com/hnt510-great/openwrt/blob/main/docs/local-build.md) for setup in a new checkout.
`install` creates build-recipe links rather than installing software on a
device. The full `update -a` also checks and can modify main-tree custom recipes.

## Maintenance and validation

`origin` points to this private repository. To merge original upstream changes
in a new clone, add `upstream` once:

```sh
git -C feeds/passwall_luci remote add upstream https://github.com/Openwrt-Passwall/openwrt-passwall.git
git -C feeds/passwall_luci fetch upstream
git -C feeds/passwall_luci merge upstream/main
# Review, resolve conflicts and validate before pushing
git -C feeds/passwall_luci push origin main
```

Skip the add command if `upstream` already exists. `scripts/feeds update` pulls
the private branch; it does not merge original upstream automatically. Feed
integration and main-tree commits are separate. Preserve existing commits,
stashes and uncommitted work. Main-tree `patches/feeds/` files and feature
manifests are historical backups; do not reapply changes already present.

Changes to published APK contents require a new package version, preserving
immutable package identities. Record source checks, package builds, VM tests
and physical-device validation separately. This page summarizes existing
design and evidence without claiming new build or device results. See
[private GitHub hosting](https://github.com/hnt510-great/openwrt/blob/main/docs/github-source-hosting.md).

## Design and validation documents

- [Switching between daed and Passwall](https://github.com/hnt510-great/openwrt/blob/main/docs/daed.md)
- [Native service supervision](https://github.com/hnt510-great/openwrt/blob/main/docs/systemd-native-supervision.md)
- [Passwall cgroup validation](https://github.com/hnt510-great/openwrt/blob/main/docs/systemd-cgroups-validation.md)
- [APK publication and source versions](https://github.com/hnt510-great/openwrt/blob/main/docs/lan-apk-feeds.md)

## License and upstream

Retain upstream copyrights and licenses. Each package's source follows its
recipe's `PKG_LICENSE` and source declarations.
See the repository [LICENSE](LICENSE).
