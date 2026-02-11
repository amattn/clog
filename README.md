# clog

v1.1.0

A simple shell utility for managing multiple [Claude Code](https://docs.anthropic.com/en/docs/claude-code) configuration directories. Switch between separate `CLAUDE_CONFIG_DIR` environments (e.g., personal, work, project-specific) with a single command.

## Why?

Claude Code stores settings, history, and project context in a config directory. By default everything lives in `~/.claude/`. If you want isolated configurations -- different settings for work vs. personal use, separate conversation histories per project, etc. -- you need to point `CLAUDE_CONFIG_DIR` at different directories.

**clog** makes that easy. It discovers all `~/.claude-*` directories on your system and lets you pick one interactively, create a new one, or quickly select one by index or name -- then optionally launch Claude Code in one step.

## Installation

### Quick Install

**Zsh** (macOS default):
```sh
mkdir -p ~/.local/bin && \
  curl -fsSL https://raw.githubusercontent.com/amattn/clog/main/clog -o ~/.local/bin/clog && \
  chmod +x ~/.local/bin/clog && \
  echo "alias clog='source ~/.local/bin/clog'" >> ~/.zshrc && source ~/.zshrc
```

**Bash** (Linux default):
```sh
mkdir -p ~/.local/bin && \
  curl -fsSL https://raw.githubusercontent.com/amattn/clog/main/clog -o ~/.local/bin/clog && \
  chmod +x ~/.local/bin/clog && \
  echo "alias clog='source ~/.local/bin/clog'" >> ~/.bashrc && source ~/.bashrc
```

> **Prefer wget?** Replace the `curl` line with:
> ```sh
> wget -qO ~/.local/bin/clog https://raw.githubusercontent.com/amattn/clog/main/clog
> ```

### Manual Install

1. **Clone the repo** (or download the `clog` script directly):

   ```sh
   git clone https://github.com/amattn/clog.git
   ```

2. **Copy the script** somewhere on your `PATH`:

   ```sh
   cp clog/clog ~/.local/bin/clog
   chmod +x ~/.local/bin/clog
   ```

   > `~/.local/bin` is just a suggestion. Use any directory on your `PATH`.

3. **Add a shell alias.** clog must be **sourced** (not executed directly) so the exported environment variable persists in your shell session. Add this line to your shell config:

   **Zsh** (`~/.zshrc`):
   ```sh
   alias clog='source ~/.local/bin/clog'
   ```

   **Bash** (`~/.bashrc`):
   ```sh
   alias clog='source ~/.local/bin/clog'
   ```

   Then reload your shell:
   ```sh
   source ~/.zshrc   # or source ~/.bashrc
   ```

### Verify

```sh
clog -v
```

You should see the version number printed, confirming everything is wired up.

## Usage

```
clog                    # Interactive menu to select or create a config
clog <number>           # Select a config by its menu index
clog <name>             # Fuzzy match by suffix (e.g., "wo" matches .claude-work)
clog -u, --unset        # Clear CLAUDE_CONFIG_DIR
clog -v, --version      # Show version
clog -h, --help         # Show help
```

### Examples

**Interactive selection** -- just run `clog` with no arguments. Select a config, then press Enter to launch Claude Code or any other key to exit:

```
$ clog
🔍 CLAUDE_CONFIG_DIR is not set. Searching for available configs...
Please select a configuration directory:
   0) Create a new config directory
   1) /Users/you/.claude-personal
   2) /Users/you/.claude-work
   x) Cancel and exit (Esc also works)
Enter your choice: 2
----------------------------------------------------------------
🚀 Environment updated!
Variable : CLAUDE_CONFIG_DIR
Value    : /Users/you/.claude-work
Status   : Active for this terminal session.

To verify or set this manually, run:
  echo $CLAUDE_CONFIG_DIR
  export CLAUDE_CONFIG_DIR="/Users/you/.claude-work"
----------------------------------------------------------------

✅ CLAUDE_CONFIG_DIR is set to: /Users/you/.claude-work
   Run 'clog -u' or 'clog --unset' to clear it.

   Enter) Launch Claude Code with config dir: .claude-work
   u) Unset CLAUDE_CONFIG_DIR
   Any other key) Exit
```

Most inputs are **single-keypress** -- no Enter required. Press `2` to select, then `Enter` to launch Claude Code, `u` to unset, or any other key to exit.

**Quick select by name** -- pass a partial, case-insensitive suffix:

```
$ clog work
```

**Quick select by number** -- pass the index directly:

```
$ clog 2
```

Both argument forms set the config and immediately show the launch prompt, so you can go straight into Claude Code.

**Already set** -- if `CLAUDE_CONFIG_DIR` is already set, clog skips the selection menu and goes straight to the launch prompt:

```
$ clog
✅ CLAUDE_CONFIG_DIR is set to: /Users/you/.claude-work
   Run 'clog -u' or 'clog --unset' to clear it.

   Enter) Launch Claude Code with config dir: .claude-work
   u) Unset CLAUDE_CONFIG_DIR
   Any other key) Exit
```

**Clear the variable** when you're done:

```
$ clog -u
🗑️  CLAUDE_CONFIG_DIR has been unset.
```

**Create a new config** -- choose option `0` from the selection menu and provide a name. clog will create `~/.claude-<name>`, set it as active, and show the launch prompt.

## How It Works

clog has two flows:

1. **Selection (Flow 1):** If `CLAUDE_CONFIG_DIR` is not set, scans `~` for `~/.claude-*` directories and presents an interactive menu. Arguments (number or partial name) skip the menu. After a selection is made, transitions to Flow 2.
2. **Launch (Flow 2):** If `CLAUDE_CONFIG_DIR` is set (either already or just selected), shows the current value and offers three single-keypress options: **Enter** to launch `claude`, **u** to unset, or **any other key** to exit.

The `u` (unset) option is only available when the script is sourced, since environment changes cannot persist from a subshell.

## Requirements

- Bash or Zsh
- `find` (standard on macOS and Linux)

## License

MIT
