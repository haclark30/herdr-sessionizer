# herdr-sessioniser

tmux-sessionizer, for [herdr](https://herdr.dev). One keybind opens an fzf
picker over your open workspaces and your zoxide directories; selecting an
open workspace focuses it, selecting a directory creates a workspace there
with your template tabs already running (default: `claude`, `nvim`, `lazygit`).

Open workspaces are ordered by how recently you switched to them, with the
current workspace last — so `prefix+f`, `enter` toggles between your two most
recent workspaces, like tmux's `switch-client -l`.

## Requirements

`herdr` ≥ 0.7, `fzf`, `zoxide`, `jq`, `bash` ≥ 4.

## Install

```sh
herdr plugin install salkhalil/herdr-sessioniser   # from GitHub
# or, from a local checkout:
herdr plugin link /path/to/herdr-sessioniser
```

Then bind a key in `~/.config/herdr/config.toml`:

```toml
[[keys.command]]
key = "prefix+f"
type = "plugin_action"
command = "sessioniser.pick"
description = "fuzzy-switch workspace"
```

and `herdr server reload-config`.

## CLI use

`bin/sessioniser` also works standalone from any shell (symlink it onto your
PATH): run it bare for the picker, or `sessioniser <dir>` to switch/create
directly — handy for scripting.

## Configuration

First run writes a commented `config.sh` to the plugin config dir
(`herdr plugin config-dir sessioniser`). Options:

- `TEMPLATE` — array of `"label:command"` tabs for new workspaces. The first
  entry replaces the initial tab and gets focus; `"label:"` leaves a plain
  shell; arguments work (`"cc:claude --continue"`).
- `GIT_ROOTS_ONLY` — set `0` to offer every zoxide directory, not just git roots.
- `EXTRA_DIRS` — directories to always offer regardless of zoxide/git.

Switch history (for recency ordering) lives in the plugin state dir.
