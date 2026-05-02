# Repository Guidelines

## Project Structure & Module Organization
- `pytomb/`: Python package code.
- `pytomb/cli.py`: Main CLI implementation (`setup`, `add`, `run`, `status`, `close`).
- `pytomb/__main__.py`: Module entrypoint for `python -m pytomb`.
- `scripts/`: Host/bootstrap shell scripts for LXD + Tor experiments.
- `docs/`: Design and ops notes (for example `docs/lxc-onion-plan.md`).
- `pyproject.toml`: Packaging metadata and console script registration.

## Build, Test, and Development Commands
- `python -m venv .venv && source .venv/bin/activate`: Create and activate local environment.
- `pip install -e .`: Install `pytomb` in editable mode.
- `pytomb status`: Check local vault state.
- `python -m pytomb setup 100`: Create a 100 MB encrypted vault.
- `python -m py_compile pytomb/*.py`: Quick syntax validation.
- `scripts/install_host_deps.sh ~/.pytomb/lxd-volume`: Install host deps for LXD/Tor topology.
- `scripts/setup_lxd_onion.sh ~/.pytomb/lxd-volume`: Bootstrap experimental onion network.

## Coding Style & Naming Conventions
- Target Python `>=3.10` and use 4-space indentation.
- Follow PEP 8 naming:
  - `snake_case` for functions/variables.
  - `UPPER_SNAKE_CASE` for module constants (for example `APP_DIR`, `VG_NAME`).
- Keep CLI errors actionable; raise `PytombError` for expected failures.
- Shell scripts should keep `set -euo pipefail` and explicit dependency checks (`need_cmd`, `command -v`).

## Testing Guidelines
- No dedicated test suite exists yet; validate changes with:
  - `python -m py_compile pytomb/*.py`
  - `pytomb status` (and relevant CLI command paths)
- For new features, add focused tests under a future `tests/` directory using `pytest` naming (`test_*.py`).
- Prioritize coverage around state handling, command execution wrappers, and error paths.

## Commit & Pull Request Guidelines
- Commit style in history is short, imperative, and capitalized (for example `Add .gitignore and stop tracking bytecode cache`).
- Keep one logical change per commit.
- PRs should include:
  - Purpose and scope.
  - Validation steps run locally.
  - Any security-sensitive behavior changes (sudo, encryption, networking).
  - Linked issue/docs update when behavior or setup changes.
