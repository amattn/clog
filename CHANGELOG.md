# Changelog

All notable changes to this project will be documented in this file.

## [1.1.0] - 2026-02-10

### Added

- Two-flow UX: Flow 1 (selection menu) transitions to Flow 2 (launch prompt) after picking a config.
- Launch Claude Code directly by pressing Enter from the launch prompt.
- Single-keypress input for most menu actions (no Enter required).
- Unset option (`u`) in the launch prompt when sourced.

### Changed

- Selection menu now uses a custom read loop instead of `select` for consistent single-keypress behavior.
- Unset helper and `u` option hidden when running in a subshell (unsourced), since unset cannot persist.
- Subshell warning now notes that `clog -u` will not work in that mode.

### Fixed

- `read -p` replaced with `printf` + `read -r` for zsh compatibility (zsh interprets `-p` as coprocess).
- `IFS=` added to single-char reads to prevent bash from stripping space/tab input.
- zsh `read -k1` returns `$'\n'` for Enter (bash returns `""`); normalized after read.
- Portable array iteration replaces direct indexing (bash 0-based vs zsh 1-based).

## [1.0.1] - 2026-02-09

### Added

- Initial release.
- Interactive menu for selecting `~/.claude-*` config directories.
- Quick selection by index number or partial name (case-insensitive).
- Create new config directories on the fly (option `0`).
- `-u`/`--unset` flag to clear `CLAUDE_CONFIG_DIR`.
- `-v`/`--version` and `-h`/`--help` flags.
- Cross-platform absolute path resolution (macOS/Linux).
