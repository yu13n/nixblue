# Nixblue

[![bluebuild build badge](https://github.com/yu13n/nixblue/actions/workflows/build.yml/badge.svg)](https://github.com/yu13n/nixblue/actions/workflows/build.yml)

Fedora Silverblue with the [Nix](https://nixos.org/) package manager preinstalled, built with [BlueBuild](https://blue-build.org/).

## Features

- **Base image**: Fedora Silverblue 44 (pinned — no surprise major-version upgrades)
- **Nix preinstalled**: `nix` and `nix-daemon` packages, daemon enabled by default
- **`/nix` on writable storage**: the root filesystem is immutable, so `/nix` is bind-mounted from `/var/nix` via the `nix.mount` systemd unit
- **Signed builds**: images are signed with cosign (Sigstore)
- **Daily rebuilds**: the image is rebuilt automatically every day at 06:00 UTC, so upstream Fedora updates land in the image within ~24 hours

## Installation

> [!WARNING]
> This is an [experimental feature](https://www.fedoraproject.org/wiki/Changes/OstreeNativeContainerStable) — try at your own discretion.

Rebase an existing atomic Fedora installation to the latest build:

1. Rebase to the unsigned image first, to get the proper signing keys and policies installed:

   ```
   rpm-ostree rebase ostree-unverified-registry:ghcr.io/yu13n/nixblue:latest
   ```

2. Reboot to complete the rebase:

   ```
   systemctl reboot
   ```

3. Rebase to the signed image:

   ```
   rpm-ostree rebase ostree-image-signed:docker://ghcr.io/yu13n/nixblue:latest
   ```

4. Reboot again to complete the installation:

   ```
   systemctl reboot
   ```

The `latest` tag always points to the newest build. The Fedora version stays pinned to the one in `recipe.yml`, so you won't be accidentally updated to the next major version.

## Updating

Updates flow through two stages:

1. **Image rebuild** — GitHub Actions rebuilds the image every day at 06:00 UTC (and on every push), picking up upstream Fedora updates.
2. **Local deployment** — the standard `rpm-ostree-autoupdate.timer` checks for new images daily and deploys them automatically; changes take effect after a reboot.

To update manually:

```
rpm-ostree upgrade
systemctl reboot
```

Check what's pending with `rpm-ostree status`. Deployments are atomic — if anything goes wrong, roll back to the previous deployment from the boot menu or with `rpm-ostree rollback`.

## Verification

Images are signed with [Sigstore](https://www.sigstore.dev/)'s [cosign](https://github.com/sigstore/cosign). Verify the signature by downloading `cosign.pub` from this repo and running:

```bash
cosign verify --key cosign.pub ghcr.io/yu13n/nixblue
```

## ISO

On Fedora Atomic you can generate an offline ISO with the instructions [here](https://blue-build.org/how-to/generate-iso/#_top). ISOs are too large to distribute on GitHub for free, so public projects need other hosting.

## License

[Apache-2.0](LICENSE)
