---
name: cmux
description: Control the cmux terminal multiplexer via its CLI/socket API. Use when user mentions "cmux", or asks to manage workspaces, panes, surfaces, send commands to terminals, automate browsers, read terminal output, show notifications, update sidebar status, or any cmux operation. Invoked via /cmux or /robert:cmux.
user-invocable: true
argument-hint: "<instruction>"
allowed-tools: Bash(cmux *), Read, Edit
---

<objective>
Execute cmux commands to manage terminal workspaces, browser automation, and sidebar metadata.
</objective>

<arguments>
$ARGUMENTS
</arguments>

<rules>
- Always run `cmux identify --json` first if you need to discover surface/workspace IDs.
- Use `cmux list-panels` to see all surfaces with their types (terminal vs browser).
- Target surfaces explicitly with `--surface surface:N` when operating on non-focused surfaces.
- For browser commands, the surface MUST be a browser surface. Use `list-panels` to find one.
- Use `--json` when you need to parse output programmatically.
- Use `cmux browser --surface surface:N screenshot --out /tmp/screenshot.png` for browser screenshots.
- Before closing workspaces, windows, or surfaces, confirm with the user unless they explicitly requested the closure.
- Read reference files from the `references/` directory when you need detailed command syntax.
</rules>

<task_routing>

**"What's on screen?" / "Take a screenshot"**
1. `cmux list-panels` → find the target surface
2. Browser surface → `cmux browser --surface surface:N screenshot --out /tmp/screenshot.png` then read the image
3. Terminal surface → `cmux read-screen --surface surface:N --scrollback --lines 100`
**"Run a command" / "Send to terminal"**
1. `cmux send --surface surface:N "command\n"`
2. Wait, then `cmux read-screen --surface surface:N` to check output

**"Open / navigate browser"**
1. `cmux browser open https://url` (new split) or `cmux browser --surface surface:N navigate https://url`
2. `cmux browser --surface surface:N wait --load-state complete`
3. `cmux browser --surface surface:N snapshot -i --compact` to inspect

**"Click / fill / interact with page"**
1. `cmux browser --surface surface:N snapshot -i --compact` to find selectors
2. `cmux browser --surface surface:N click "#selector"` / `fill` / `type` etc.
3. Use `--snapshot-after` to verify result inline

**"Set up workspace layout"**
1. `cmux new-workspace` / `cmux new-split right|down` / `cmux new-pane`
2. `cmux rename-workspace "name"` / `cmux rename-tab "name"`
3. `cmux send --surface surface:N "command\n"` to set up each terminal

**"Show progress / status"**
1. `cmux set-status key "value" --icon name --color "#hex"`
2. `cmux set-progress 0.5 --label "Building..."`
3. `cmux log --level success -- "message"`

</task_routing>

<object_model>

```
Window (OS window)
└── Workspace (sidebar entry)
    └── Pane (split region)
        └── Surface (tab: terminal or browser)
```

**Addressing:** `window:1`, `workspace:2`, `pane:3`, `surface:4` (short refs — preferred), UUIDs, or numeric indexes.

**Environment vars** (auto-set in cmux terminals): `CMUX_WORKSPACE_ID`, `CMUX_SURFACE_ID`, `CMUX_TAB_ID`. Commands default to caller's context when flags are omitted.

</object_model>

<essential_commands>

```sh
# Discovery
cmux identify --json                                       # caller + focused surface
cmux list-workspaces                                       # workspaces in window
cmux list-panes                                            # panes in workspace
cmux list-panels                                           # surfaces with types

# Terminal I/O
cmux send --surface surface:N "command\n"                  # send text (\n = enter)
cmux send-key --surface surface:N ctrl+c                   # send key event
cmux read-screen --surface surface:N --scrollback --lines 200

# Browser core
cmux browser open https://url                              # new browser split
cmux browser --surface surface:N navigate https://url
cmux browser --surface surface:N screenshot --out /tmp/s.png
cmux browser --surface surface:N snapshot -i --compact     # interactive DOM
cmux browser --surface surface:N click "#selector"
cmux browser --surface surface:N fill "#selector" "text"
cmux browser --surface surface:N wait --load-state complete
cmux browser --surface surface:N get title
cmux browser --surface surface:N eval "document.title"

# Layout
cmux new-workspace                                         # new workspace
cmux new-split right|down                                  # split pane
cmux rename-workspace "name"
cmux select-workspace --workspace workspace:N

# Sidebar
cmux set-status key "value" --icon name --color "#hex"
cmux set-progress 0.5 --label "Building..."
cmux log --level info|success|warning|error -- "message"
cmux notify --title "Done" --body "message"
```

</essential_commands>

<reference_files>
For detailed syntax beyond the essentials above, read the relevant file:

- `references/layout.md` — windows, workspaces, panes, surfaces, tabs
- `references/terminal.md` — send, send-key, read-screen, pipe-pane
- `references/browser.md` — full browser automation (screenshot, snapshot, interaction, wait, extraction, cookies, storage, frames, tabs)
- `references/sidebar.md` — status pills, progress bar, log entries, notifications
- `references/advanced.md` — system commands, sync/wait-for, search, buffers, hooks, claude integration
</reference_files>

<self_improvement>
This skill's base directory is: ${CLAUDE_SKILL_DIR}

When a cmux command fails and the cause is **incorrect or missing information in this skill** (wrong flags, missing subcommand, outdated syntax), fix the skill:

1. Diagnose: Is the failure due to a skill documentation error, or a runtime/user error?
   - Skill error: command syntax wrong, flag doesn't exist, subcommand missing from references, incorrect usage pattern
   - NOT a skill error: wrong surface targeted, network issues, app not running, user typo
2. Fix: Edit the relevant file (SKILL.md or `references/*.md`) to correct the information.
   - Wrong syntax → fix the example
   - Missing command → add it to the appropriate reference file and to essential_commands if commonly used
   - Incorrect flag → update the flag
3. Continue: Retry the command with the corrected syntax after updating the skill.

Do NOT update skill files for runtime errors, permission issues, or user mistakes.
</self_improvement>

<success_criteria>
- The requested cmux operation completed without error
- Output was verified (read-screen after send, snapshot after browser interaction, screenshot read after capture)
- User-visible state matches the intent of the instruction
- If a skill documentation error was encountered, the relevant file was updated
</success_criteria>
