# clog

A simple shell utility for managing multiple [Claude Code](https://docs.anthropic.com/en/docs/claude-code) configuration directories. Switch between separate `CLAUDE_CONFIG_DIR` environments (e.g., personal, work, project-specific) with a single command.

## Why?

Claude Code stores settings, history, and project context in a config directory. By default everything lives in `~/.claude/`. If you want isolated configurations -- different settings for work vs. personal use, separate conversation histories per project, etc. -- you need to point `CLAUDE_CONFIG_DIR` at different directories.

**clog** makes that easy. It discovers all `~/.claude-*` directories on your system and lets you create a new one, pick one interactively, or quickly select one with a short argument by index or by name.  

## Installation

### Quick Install

**Zsh** (macOS default):
```sh
mkdir -p ~/bin && \
  curl -fsSL https://raw.githubusercontent.com/amattn/clog/main/clog -o ~/bin/clog && \
  chmod +x ~/bin/clog && \
  echo "alias clog='source ~/bin/clog'" >> ~/.zshrc && source ~/.zshrc
```

**Bash** (Linux default):
```sh
mkdir -p ~/bin && \
  curl -fsSL https://raw.githubusercontent.com/amattn/clog/main/clog -o ~/bin/clog && \
  chmod +x ~/bin/clog && \
  echo "alias clog='source ~/bin/clog'" >> ~/.bashrc && source ~/.bashrc
```

> **Prefer wget?** Replace the `curl` line with:
> ```sh
> wget -qO ~/bin/clog https://raw.githubusercontent.com/amattn/clog/main/clog
> ```

### Manual Install

1. **Clone the repo** (or download the `clog` script directly):

   ```sh
   git clone https://github.com/amattn/clog.git
   ```

2. **Copy the script** somewhere on your `PATH`:

   ```sh
   cp clog/clog ~/bin/clog
   chmod +x ~/bin/clog
   ```

   > `~/bin` is just a suggestion. Use any directory on your `PATH`.

3. **Add a shell alias.** clog must be **sourced** (not executed directly) so the exported environment variable persists in your shell session. Add this line to your shell config:

   **Zsh** (`~/.zshrc`):
   ```sh
   alias clog='source ~/bin/clog'
   ```

   **Bash** (`~/.bashrc`):
   ```sh
   alias clog='source ~/bin/clog'
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

**Interactive selection** -- just run `clog` with no arguments:

```
$ clog
CLAUDE_CONFIG_DIR is not set. Searching for available configs...
Please select a configuration directory:
0) Create a new config directory
1) /Users/you/.claude-personal
2) /Users/you/.claude-work
3) Cancel and exit (x)
Enter the number of your choice (or 'x' to cancel): 2
----------------------------------------------------------------
Environment updated!
Variable : CLAUDE_CONFIG_DIR
Value    : /Users/you/.claude-work
Status   : Active for this terminal session.

To verify or set this manually, run:
  echo $CLAUDE_CONFIG_DIR
  export CLAUDE_CONFIG_DIR="/Users/you/.claude-work"
----------------------------------------------------------------
```

**Quick select by name** -- pass a partial, case-insensitive suffix:

```
$ clog work
```

**Quick select by number** -- pass the index directly:

```
$ clog 2
```

**Clear the variable** when you're done:

```
$ clog -u
CLAUDE_CONFIG_DIR has been unset.
```

**Create a new config** -- choose option `0` from the interactive menu and provide a name. clog will create `~/.claude-<name>` and set it as active.

## How It Works

1. Checks if `CLAUDE_CONFIG_DIR` is already set. If so, reports the current value and reminds you how to unset it.
2. Scans your home directory for directories matching `~/.claude-*`.
3. If an argument is provided, attempts to match it as an index or a partial name.
4. If no match is found (or no argument given), presents an interactive menu.
5. Exports `CLAUDE_CONFIG_DIR` for the current terminal session.

## Requirements

- Bash or Zsh
- `find` (standard on macOS and Linux)

## License

MIT
