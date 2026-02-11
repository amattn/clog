# clog - Product Requirements Document

v1.1.0

## Overview

**clog** is a shell utility for managing multiple Claude Code configuration directories. It enables users to switch between isolated `CLAUDE_CONFIG_DIR` environments (e.g., personal, work, project-specific) from a single interactive command.

## Problem Statement

Claude Code stores settings, conversation history, and project context in a config directory (default: `~/.claude/`). Users who want isolated configurations -- different settings for work vs. personal, separate histories per project, sandboxed experiments -- must manually manage the `CLAUDE_CONFIG_DIR` environment variable. This involves remembering directory paths, typing export commands, and unsetting when done. The process is error-prone and tedious.

## Target Users

- Developers using Claude Code across multiple projects or contexts
- Users who want separate conversation histories or settings per environment
- Teams where members need isolated Claude Code configurations

## User Flows

### Flow 1: Selection Menu (CLAUDE_CONFIG_DIR is not set)

This flow is entered when `CLAUDE_CONFIG_DIR` is not set. It discovers `~/.claude-*` directories and presents a selection menu.

**Display:**

```
🔍 CLAUDE_CONFIG_DIR is not set. Searching for available configs...
Please select a configuration directory:
0) Create a new config directory
1) /Users/you/.claude-personal
2) /Users/you/.claude-work
x) Cancel and exit (Esc also works)
Enter your choice:
```

**Input (single-keypress where possible, no Enter required):**

| Input | Action | Requires Enter? |
|-------|--------|-----------------|
| `0` | Create a new config directory (prompt for name, create `~/.claude-<name>`) | No |
| `1`-`9` | Select an existing directory (when < 10 entries) | No |
| `1`-`9`... | Begin multi-digit entry (when >= 10 entries) | Yes |
| `x` / `Esc` | Cancel without setting the variable | No |
| `exit` | Cancel without setting the variable | Yes |

**After selection:** Once a directory is selected or created (options `0` through `N`), the script exports `CLAUDE_CONFIG_DIR` and **immediately transitions to Flow 2**. This gives the user a chance to launch Claude Code right away or exit.

#### Quick Selection via Arguments

Flow 1 also supports non-interactive selection via positional arguments:

- **Numeric argument** (`clog 2`): Treat as 1-based index into the discovered directory list
- **String argument** (`clog work`): Case-insensitive partial match against directory suffixes
- If exactly one match is found, set it and transition to Flow 2 (no menu)
- If multiple matches, warn and fall through to the interactive menu
- If no match, warn and fall through to the interactive menu

### Flow 2: Launch Prompt (CLAUDE_CONFIG_DIR is set)

This flow is entered when `CLAUDE_CONFIG_DIR` is already set -- either because it was set before clog was invoked, or because Flow 1 just set it.

**Display (when sourced):**

```
✅ CLAUDE_CONFIG_DIR is set to: /Users/you/.claude-work
   Run 'clog -u' or 'clog --unset' to clear it.

   Enter) Launch Claude Code with config dir: .claude-work
   u) Unset CLAUDE_CONFIG_DIR
   Any other key) Exit
```

**Display (when not sourced):** The unset helper line and `u)` menu item are hidden, since unset cannot persist outside the subshell.

```
✅ CLAUDE_CONFIG_DIR is set to: /Users/you/.claude-work

   Enter) Launch Claude Code with config dir: .claude-work
   Any other key) Exit
```

**Input (all single-keypress, no Enter required):**

| Input | Action |
|-------|--------|
| `Enter` | Launch `claude` with the current config directory |
| `u` | Unset `CLAUDE_CONFIG_DIR` and exit (only when sourced; ignored otherwise) |
| Any other key | Exit without changes |

- Single-keypress detection uses `IFS= read -rsn1` (bash) / `IFS= read -rsk1` (zsh) for immediate response

### Create New Config Directory

- Accessible as option `0` from Flow 1
- Prompt for a descriptive name (validated non-empty; re-prompts on empty input)
- Create `~/.claude-<name>` via `mkdir -p`
- Export as `CLAUDE_CONFIG_DIR` and transition to Flow 2

### Unset Support

- `clog -u` or `clog --unset` flag: Immediately unset `CLAUDE_CONFIG_DIR` and exit (bypasses both flows)
- `u` key in Flow 2: Unset and exit (immediate, no Enter required). Only available when sourced -- hidden and ignored when running in a subshell, since unset cannot persist outside the subshell.

## Other Requirements

### Source Detection

- Detect whether the script was sourced (`source clog` / `. clog`) or executed directly (`./clog`)
- If not sourced, warn the user that:
  - The variable will only persist in the subshell
  - Unset (`clog -u`) will not work in this mode
- Use `ZSH_EVAL_CONTEXT` (zsh) and `BASH_SOURCE` (bash) for detection
- Source status affects UI: unset options are hidden in Flows 1 and 2 when not sourced

### Confirmation Output

When transitioning from Flow 1 to Flow 2 after a new selection (Flow 1 → Flow 2), display a confirmation block before the Flow 2 prompt:

```
----------------------------------------------------------------
🚀 Environment updated!
Variable : CLAUDE_CONFIG_DIR
Value    : /Users/you/.claude-work
Status   : Active for this terminal session.

To verify or set this manually, run:
  echo $CLAUDE_CONFIG_DIR
  export CLAUDE_CONFIG_DIR="/Users/you/.claude-work"
----------------------------------------------------------------
```

- Status reflects whether the script was sourced or run in a subshell
- Only shown when the value was newly set or changed

## Command-Line Interface

```
clog                    # Interactive menu
clog <number>           # Select by index
clog <suffix>           # Fuzzy match by name
clog -u, --unset        # Clear CLAUDE_CONFIG_DIR
clog -v, --version      # Show version
clog -h, --help         # Show help with alias setup instructions
```

## Cross-Platform Requirements

- Must work when sourced in both **bash** and **zsh**
- Shell-specific portability concerns:
  - `read -p` is not portable (means "coprocess" in zsh); use `printf` + `read -r` instead
  - `read -n1` (bash) vs `read -k1` (zsh) for single-character reads
  - `IFS=` must prefix single-char reads to prevent bash from stripping space/tab (default IFS characters) from the input value
  - zsh `read -k1` returns `$'\n'` for Enter (literal newline); bash `read -n1` returns `""` (newline is delimiter). Normalize after read: `[[ "$var" == $'\n' ]] && var=""`
  - Array indexing differences (0-based in bash, 1-based in zsh); use iteration instead of direct indexing
  - `realpath` may not exist on older macOS; fall back to `cd`/`pwd`/`basename`
- Shebang is `#!/bin/bash` but irrelevant when sourced (host shell interprets the script)

## Non-Functional Requirements

- **Zero dependencies**: Only requires `find` (standard on macOS and Linux)
- **Single file**: Entire utility is one shell script
- **Namespace safety**: Internal variables use `_clog_` prefix to avoid polluting the user's shell environment
- **No persistent state**: No config files, databases, or caches; all state comes from the filesystem (`~/.claude-*` directories) and environment (`CLAUDE_CONFIG_DIR`)
- **Non-destructive**: Never deletes directories or modifies files outside its own scope

## Out of Scope

- Managing the contents of config directories
- Syncing configurations between machines
- Integration with Claude Code internals
- Auto-updating or package manager distribution
- Config directory naming conventions beyond `~/.claude-*`
