# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

- `python -m venv .venv && source .venv/bin/activate && pip install -e .` — set up the local environment and install `pytomb` in editable mode.
- `python -m py_compile pytomb/*.py` — quick syntax validation. There is no automated test suite yet; validate CLI changes manually with `pytomb status` and the relevant command path.
- `python -m pytomb setup 100` — create a 100 MB encrypted vault for local validation.
- `pytomb status` — check local vault state.
- `scripts/install_host_deps.sh ~/.pytomb/lxd-volume` — install host deps for the LXD/Tor topology.
- `scripts/setup_lxd_onion.sh ~/.pytomb/lxd-volume` — bootstrap the experimental onion network.

## Architecture

`pytomb` is a single-file CLI (`pytomb/cli.py`) with five subcommands: `setup`, `add`, `run`, `status`, `close`.

**Storage stack:** loop image (`~/.pytomb/driveN.img`) -> `losetup` -> LUKS (`cryptsetup`) -> LVM PV/VG/LV -> ext4 -> mounted at `~/.pytomb/mnt`.

**Key management:** a 64-byte random master key is stored as `~/.pytomb/keys/master.key.gpg`. If a GPG secret key exists, the master key is encrypted asymmetrically with the first available secret key fingerprint; otherwise it is encrypted symmetrically with a user passphrase. The `decrypted_master_key()` context manager decrypts it to a temporary file, yields the path, and cleans up in `finally`.

**State tracking:** `~/.pytomb/state.json` is a JSON file tracking image indices, file names, and LUKS mapping names. `load_state()` and `save_state()` are the only readers/writers.

**Firefox integration:** `run` opens all LUKS mappings, activates LVM, mounts the volume, bootstraps a Firefox profile inside the mount (with `user.js` prefs), downloads Bitwarden and NoScript add-ons on first run, and launches `firefox -no-remote -profile <mount-path>`.

**Command execution:** `run_cmd()` is the central subprocess wrapper. It automatically prepends `sudo` when the process is not running as root (`os.geteuid() != 0`). Expected failures are raised as `PytombError` with actionable messages.

**No external Python dependencies:** the project uses only the standard library (`argparse`, `subprocess`, `json`, `urllib.request`, `pathlib`, `tempfile`, etc.).

## Coding Style

- Target Python `>=3.10` with 4-space indentation.
- PEP 8 naming: `snake_case` for functions/variables, `UPPER_SNAKE_CASE` for module constants.
- Shell scripts must use `set -euo pipefail` and explicit dependency checks (`need_cmd`, `command -v`).

## Commit and PR Style

- Short, imperative, capitalized commit messages (e.g. `Add .gitignore and stop tracking bytecode cache`).
- One logical change per commit.
- PRs should include purpose/scope, local validation steps, and flag any security-sensitive behavior changes (sudo, encryption, networking).

## System Dependencies

- `sudo`, `gpg`, `losetup`, `cryptsetup`, `lvm2`, `fallocate`, `findmnt`, `firefox`

## Experimental LXC + Tor Isolation

- Design doc: `docs/lxc-onion-plan.md`
- Host deps installer: `scripts/install_host_deps.sh`
- Topology bootstrap: `scripts/setup_lxd_onion.sh`
