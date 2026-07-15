# herdr-sessionizer

tmux-sessionizer, for [herdr](https://herdr.dev). One keybind opens an fzf
picker over your open workspaces and your zoxide directories; selecting an
open workspace focuses it, selecting a directory creates a workspace there
with your template tabs already running (default: `claude`, `nvim`, `lazygit`).

Open workspaces are ordered by how recently you switched to them, with the
current workspace last — so `prefix+f`, `enter` toggles between your two most
recent workspaces, like tmux's `switch-client -l`.

## Requirements

`herdr` ≥ 0.7, `fzf`, `jq`, `bash` ≥ 4. `zoxide` is recommended but optional —
without it the picker only offers open workspaces and `extra_dirs`.

## Install

```sh
herdr plugin install salkhalil/herdr-sessionizer   # from GitHub
# or, from a local checkout:
herdr plugin link /path/to/herdr-sessionizer
```

Then bind a key in `~/.config/herdr/config.toml`:

```toml
[[keys.command]]
key = "prefix+f"
type = "plugin_action"
command = "sessionizer.pick"
description = "fuzzy-switch workspace"
```

and `herdr server reload-config`.

## CLI use

`bin/sessionizer` also works standalone from any shell (symlink it onto your
PATH): run it bare for the picker, or `sessionizer <dir>` to switch/create
directly — handy for scripting.

## Configuration

First run writes `config.json` to the plugin config dir
(`herdr plugin config-dir sessionizer`):

```json
{
  "template": [
    { "tab": "cc", "command": "claude" },
    { "tab": "vim", "command": "nvim" },
    { "tab": "lazygit", "command": "lazygit" }
  ],
  "git_roots_only": true,
  "extra_dirs": ["~/Documents/some-project"],
  "overrides": [
    {
      "match": "*/Tortus/*",
      "template": [
        { "tab": "cc", "command": "claude" },
        { "tab": "dev", "command": "pnpm dev" },
        { "tab": "scratch" }
      ]
    }
  ]
}
```

- `template` — tabs for new workspaces. The first entry replaces the initial
  tab and gets focus; `command` is typed into the tab's shell (arguments
  work), and omitting it leaves a plain shell.
- `git_roots_only` — set `false` to offer every zoxide directory, not just
  git roots.
- `extra_dirs` — directories to always offer regardless of zoxide/git
  (leading `~` is expanded).
- `overrides` — per-project templates: the first entry whose glob `match`es
  the new workspace's directory replaces `template`. Overrides live in your
  config, never in the target repo, so cloned code can't inject commands
  into fresh shells.

Switch history (for recency ordering) lives in the plugin state dir.
