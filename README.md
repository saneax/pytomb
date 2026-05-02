# pytomb

`pytomb` creates an encrypted local storage vault using loopback image files, LUKS encryption, and LVM, then launches Firefox with a browser profile stored inside the mounted vault.

## Installation

Requires Python >=3.10. No external Python dependencies.

```bash
pip install .
```

Or in editable mode for development:

```bash
pip install -e .
```

## System Dependencies

Install these tools before running `pytomb`:

- `sudo`
- `gpg`
- `losetup` (util-linux)
- `cryptsetup`
- `lvm2` (`pvcreate`, `vgcreate`, `lvcreate`, `lvchange`, `vgchange`, ...)
- `fallocate`
- `findmnt`
- `firefox`

Most storage operations require root privileges and are executed through `sudo`.

## Quickstart

```bash
# Create a 100 MB encrypted vault
pytomb setup

# Add another 200 MB image to expand the vault
pytomb add 200

# Mount the vault and launch Firefox with the vault profile
pytomb run

# Check vault status
pytomb status

# Unmount and close everything
pytomb close
```

## Commands

- `pytomb setup [size_mb]` — Create the first encrypted loopback image, initialize the LUKS mapping, create the LVM volume group and logical volume, format it as ext4, and mount it. Defaults to 100 MB if size is omitted.
- `pytomb add [size_mb]` — Create an additional encrypted image, add its LUKS mapping to the existing LVM volume group, extend the logical volume, and resize the filesystem. Defaults to 100 MB if size is omitted.
- `pytomb run` — Unlock all configured images, activate the volume group, mount the logical volume, bootstrap the Firefox profile (if needed), and launch Firefox.
- `pytomb status` — Show the current state of images, loop devices, LUKS mappings, LVM volumes, and mount status.
- `pytomb close` — Unmount the volume, deactivate the logical volume and volume group, close all LUKS mappings, and detach loopback devices.

## Storage Layout

- Base directory: `~/.pytomb`
- Encrypted images: `~/.pytomb/drive1.img`, `drive2.img`, ...
- Mount point: `~/.pytomb/mnt`
- Encrypted master key: `~/.pytomb/keys/master.key.gpg`
- Key metadata: `~/.pytomb/keys/gpg_meta.json`
- State file: `~/.pytomb/state.json`

## Key Handling

- If a GPG secret key exists, the master key is encrypted asymmetrically with your first available secret key fingerprint.
- If no GPG secret key exists, `pytomb` prompts for a symmetric passphrase and encrypts the master key with it (AES256).
- The master key is a 64-byte random blob used as the LUKS key file for all images.

## Firefox Profile

When `pytomb run` is executed:

1. The vault is mounted at `~/.pytomb/mnt`.
2. A Firefox profile directory is created inside the mount at `firefox-profile/`.
3. A `user.js` with privacy-oriented defaults is written:
   - disables startup page and new tab page
   - disables search suggestions in the URL bar
   - enables tracking protection and DNT header
   - disables password remember prompts
4. Firefox add-ons are downloaded from Mozilla on first run:
   - Bitwarden Password Manager
   - NoScript Security Suite

If add-on download fails (offline or blocked network), the profile is still created and Firefox runs without the add-ons.

## LXC + Tor Isolation (experimental)

An optional LXD topology is included to run a browser node behind a Tor gateway container.

- Plan: `docs/lxc-onion-plan.md`
- Host dependency installer: `scripts/install_host_deps.sh`
- Topology bootstrap: `scripts/setup_lxd_onion.sh`

Example:

```bash
scripts/install_host_deps.sh ~/.pytomb/lxd-volume
scripts/setup_lxd_onion.sh ~/.pytomb/lxd-volume
```
