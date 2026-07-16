# herdr CLI reference (as of herdr 0.7.3)

Working notes for sessionizer development. Everything below verified live
against a running server. All commands return JSON envelopes:
`{"id": "cli:...", "result": {...}}` — payload is always under `.result`.

## Agent state model

`agent_status` values: **`idle` | `working` | `blocked` | `done` | `unknown`**

- `blocked` — agent is waiting on user input (permission prompt, question).
- `done` — terminal state reported by some integrations; `wait agent-status`
  accepts it, `pane report-agent` does not (it takes idle|working|blocked|unknown).
- `unknown` — no agent detected in the pane. Non-agent tabs (vim, lazygit,
  plain shells) report `unknown`.
- Status rolls up: pane → tab → workspace (each tab/workspace carries an
  aggregated `agent_status`).
- Detection is via per-agent manifests (OSC title rules etc.);
  `herdr agent explain <target>` shows which rule matched and its evidence.

## Snapshot (what the sessionizer already uses)

```sh
herdr api snapshot    # full live state: .result.snapshot.{workspaces,tabs,panes}
```

- workspace: `workspace_id, label, number, focused, agent_status, active_tab_id, tab_count, pane_count`
- tab: `tab_id ("w1:t2"), workspace_id, label, number, focused, agent_status, pane_count`
- pane: `pane_id ("w1:p1"), tab_id, workspace_id, terminal_id, cwd, foreground_cwd, focused, agent, agent_status, scroll, revision`

`herdr api schema [--json]` dumps the full socket API schema.

## Agents — the interesting surface for the agent view

```sh
herdr agent list
```
Returns `.result.agents[]` — **only panes with a detected agent**, each row:
`agent` (e.g. "claude"), `agent_status`, `pane_id`, `tab_id`, `workspace_id`,
`terminal_id`, `cwd`, `foreground_cwd`, `focused`. This is the complete data
source for an agent-centric picker; no per-workspace fan-out needed.

```sh
herdr agent focus <target>
```
Focuses workspace + tab + pane in ONE command (verified: cross-workspace jump
works). Targets accept pane ids, terminal ids, unique agent names, or agent labels.

Other agent commands:

```sh
herdr agent get <target>                 # one agent's row
herdr agent read <target> [--source visible|recent|recent-unwrapped] [--lines N]
                                         # scrollback text — preview material
herdr agent send <target> <text>         # literal text (no Enter); use pane run for cmd+Enter
herdr agent rename <target> <name>       # custom agent name (also --clear)
herdr agent wait <target> --status <s> [--timeout MS]
herdr agent explain <target>             # which detection rule fired + evidence
herdr agent start <name> [--cwd ...] [--workspace ID] [--tab ID] [--split right|down] -- <argv...>
```

## Workspaces / tabs

```sh
herdr workspace list|get|focus|rename|close
herdr workspace create [--cwd PATH] [--label TEXT] [--focus|--no-focus]
  # create returns .result.{workspace.workspace_id, tab.tab_id, root_pane.pane_id}

herdr tab list [--workspace <ws_id>]     # .result.tabs[] with agent_status
herdr tab create [--workspace ID] [--cwd PATH] [--label TEXT] [--focus|--no-focus]
herdr tab get|focus|rename|close <tab_id>
```

`tab focus` alone does NOT switch workspace context for the picker's purposes —
prefer `agent focus` for agent jumps; for non-agent tabs use
`workspace focus` + `tab focus`.

## Panes

```sh
herdr pane list [--workspace ID]
herdr pane run <pane_id> <command>       # types command + Enter (sessionizer uses this)
herdr pane send-text <pane_id> <text>    # literal text, no Enter
herdr pane send-keys <pane_id> <key>...
herdr pane read <pane_id> [--lines N]    # scrollback
herdr pane focus --direction left|right|up|down   # directional only — no focus-by-id!
herdr pane split|swap|move|zoom|resize|close|rename
```

Note: there is no `pane focus <id>`. Focus-by-id exists only at workspace, tab,
and agent level.

## Blocking waits & notifications

```sh
herdr wait agent-status <pane_id> --status <idle|working|blocked|done|unknown> [--timeout MS]
herdr wait output <pane_id> --match <text> [--regex] [--timeout MS]
herdr notification show <title> [--body TEXT] [--sound none|done|request]
```

## Sessionizer-relevant conclusions

- Agent view = `herdr agent list` → sort by status priority → `herdr agent focus <pane_id>` on select. Two commands total.
- Suggested status priority: `blocked` > `done` > `working` > `idle` (`unknown` never appears in `agent list`).
- `agent read` gives scrollback for an fzf preview of what the agent is doing.
- Tab-level rows for non-agent tabs would come from snapshot `.tabs[]`, jump = `workspace focus` + `tab focus`.
