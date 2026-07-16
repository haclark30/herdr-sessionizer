# herdr-sessionizer — Context

The sessionizer is a herdr plugin: one keybind opens an fzf picker over open
workspaces and zoxide directories; selecting focuses or creates a workspace
with template tabs. See `README.md` for user-facing docs and
`docs/herdr-cli.md` for the herdr CLI surface it builds on.

## Glossary

- **Workspace picker** — the default picker view: open workspaces (recency-ordered,
  focused workspace last) followed by zoxide/extra directories. Selecting an open
  workspace focuses it; selecting a directory creates a workspace from the template.
- **Agent view** — the second picker view: one row per *agent* (a pane where herdr
  detected a running agent), ordered most-urgent first. Selecting a row jumps
  straight to that agent via `herdr agent focus`. It is an agents-only inbox —
  non-agent tabs (vim, lazygit, plain shells) never appear; those are reached
  through the workspace picker. Deliberately NOT a general tab list (decided
  2026-07-15: tabs without agents are noise, need a second jump path, and always
  sort last anyway). Entered two ways: Tab flips between the two views inside
  the picker (stateless fzf `transform` on `$FZF_PROMPT`), and a dedicated
  `sessionizer.agents` action opens the picker already in agent view.
- **Agent-view ordering** — status priority `blocked > done > working > idle`,
  then workspace number, then tab number (stable/positional — rows don't
  reshuffle between opens). The currently focused agent always sorts last,
  mirroring the workspace picker's focused-last convention, so plain `enter`
  jumps to the most urgent agent *elsewhere*.
- **Agent status** — herdr's per-pane state: `blocked` (waiting on user input),
  `done`, `working`, `idle`, `unknown` (no agent detected; never appears in the
  agent view). Rolls up pane → tab → workspace.
- **Agent-view row** — status-first columns: ANSI-colored status word
  (blocked=red, done=green, working=yellow, idle=dim), then `workspace · tab`,
  then agent name. The preview pane shows the agent's live screen (header with
  agent · status · cwd, then `herdr agent read --source visible --format ansi`)
  so a blocked agent's actual prompt is visible before jumping.
- **Template** — the tab/command list applied to newly created workspaces
  (default cc/claude, vim/nvim, lazygit/lazygit), overridable per directory glob.
- **Switch history** — cwd-keyed append log in the plugin state dir; drives
  recency ordering of open workspaces in the workspace picker. Agent-view jumps
  record too (decided 2026-07-15: a jump is a real workspace switch, and skipping
  it would leave enter-toggle stale) — keyed by the target *workspace's*
  first-pane cwd, the same key the workspace picker uses, not the agent pane's
  own cwd (which may be a subdirectory).
