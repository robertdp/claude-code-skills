# Layout: Windows, Workspaces, Panes, Surfaces

## Windows

```sh
cmux list-windows                                          # list OS windows
cmux current-window                                        # focused window
cmux new-window                                            # create window
cmux focus-window --window window:1                        # bring to front
cmux close-window --window window:2                        # close window
cmux next-window | previous-window | last-window           # navigate (tmux compat)
```

## Workspaces

```sh
cmux list-workspaces                                       # list in current window
cmux current-workspace                                     # current workspace UUID
cmux new-workspace                                         # create workspace
cmux new-workspace --command "htop"                        # create with command
cmux select-workspace --workspace workspace:2              # switch to workspace
cmux close-workspace --workspace workspace:3               # close workspace
cmux rename-workspace "backend logs"                       # rename (defaults to current)
cmux rename-workspace --workspace workspace:2 "agent run"
```

### workspace-action

Actions: `pin`, `unpin`, `rename`, `clear-name`, `move-up`, `move-down`, `move-top`, `close-others`, `close-above`, `close-below`, `mark-read`, `mark-unread`

```sh
cmux workspace-action --action pin
cmux workspace-action --workspace workspace:2 --action pin
cmux workspace-action --action rename --title "infra"
```

### reorder / move

```sh
cmux reorder-workspace --workspace workspace:2 --index 0
cmux reorder-workspace --workspace workspace:3 --after workspace:1
cmux move-workspace-to-window --workspace workspace:2 --window window:1
```

## Panes

```sh
cmux list-panes                                            # panes in current workspace
cmux list-panes --workspace workspace:2
cmux focus-pane --pane pane:2
cmux new-split right                                       # split current pane
cmux new-split down --workspace workspace:2
cmux new-pane                                              # terminal, split right
cmux new-pane --direction down
cmux new-pane --type browser --url https://example.com     # browser pane
cmux swap-pane --pane pane:1 --target-pane pane:2
cmux break-pane                                            # surface -> new workspace
cmux break-pane --surface surface:2 --no-focus
cmux join-pane --target-pane pane:2                        # surface -> existing pane
cmux last-pane                                             # toggle last focused pane
# Note: resize-pane exists but returns not_supported
```

## Surfaces (Tabs)

```sh
cmux list-pane-surfaces                                    # surfaces in current pane
cmux list-panels                                           # all surfaces with type info
cmux new-surface                                           # new terminal tab
cmux new-surface --type browser --url https://example.com  # new browser tab
cmux new-surface --pane pane:2                             # in specific pane
cmux close-surface --surface surface:3
cmux focus-panel --panel surface:2                         # focus a surface
cmux move-surface --surface surface:1 --workspace workspace:2
cmux move-surface --surface surface:1 --pane pane:2 --index 0
cmux reorder-surface --surface surface:1 --index 0
cmux drag-surface-to-split --surface surface:1 right       # surface -> new split
```

### Tab actions

Actions: `rename`, `clear-name`, `close-left`, `close-right`, `close-others`, `new-terminal-right`, `new-browser-right`, `reload`, `duplicate`, `pin`, `unpin`, `mark-read`, `mark-unread`

```sh
cmux rename-tab "build logs"
cmux tab-action --tab tab:3 --action pin
cmux tab-action --action new-browser-right --url https://example.com
```

### Misc surface commands

```sh
cmux surface-health                                        # check surface health
cmux trigger-flash --surface surface:2                     # visual flash
cmux refresh-surfaces                                      # force refresh all
cmux clear-history --surface surface:2                     # clear scrollback
cmux respawn-pane --surface surface:2 --command "htop"     # restart terminal
```
