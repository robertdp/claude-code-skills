# Advanced: Sync, Buffers, Search, Hooks, Integration

## System

```sh
cmux version
cmux ping                                                  # check socket
cmux capabilities                                          # list all methods (JSON)
cmux identify                                              # caller + focused info (JSON)
cmux identify --no-caller                                  # focused only
```

## Synchronization

```sh
cmux wait-for my-signal --timeout 30                       # block until signalled
cmux wait-for --signal my-signal                           # signal waiters
cmux wait-for -S my-signal                                 # shorthand
```

## Search

```sh
cmux find-window "build"                                   # search by title
cmux find-window --content "ERROR"                         # search terminal content
cmux find-window --select "logs"                           # find and switch
```

## Buffers (tmux compat)

```sh
cmux set-buffer "copied text"
cmux set-buffer --name my-buffer "stored text"
cmux list-buffers
cmux paste-buffer
cmux paste-buffer --name my-buffer --surface surface:2
```

## Hooks (tmux compat)

```sh
cmux set-hook --list
cmux set-hook <event> <command>
cmux set-hook --unset <event>
```

## App focus

```sh
cmux set-app-focus active | inactive | clear
cmux simulate-app-active
```

## Claude Code integration

Reads JSON from stdin.

```sh
echo '{"session_id":"abc"}' | cmux claude-hook session-start
echo '{}' | cmux claude-hook stop
echo '{"message":"Done"}' | cmux claude-hook notification
```

## Display (tmux compat)

```sh
cmux display-message "Hello world"
cmux display-message -p "Hello world"                      # print to stdout
```

## Misc stubs (tmux compat)

`popup`, `bind-key`, `unbind-key`, `copy-mode` — compatibility stubs.
