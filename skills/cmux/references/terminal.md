# Terminal I/O

## Send text

Escape sequences: `\n`/`\r` = Enter, `\t` = Tab. Use `--` before text starting with `-`.

```sh
cmux send "echo hello\n"                                   # type + enter
cmux send --surface surface:2 "ls -la\n"                   # to specific surface
cmux send -- "-flag-like-text\n"                           # text starting with -
cmux send-panel --panel surface:2 "echo hello\n"           # by panel ref
```

## Send keys

Keys: `enter`, `tab`, `escape`, `backspace`, `delete`, `up`, `down`, `left`, `right`
Modifiers: `ctrl+c`, `ctrl+z`, `ctrl+d`, etc.

```sh
cmux send-key enter
cmux send-key ctrl+c
cmux send-key --surface surface:2 ctrl+c
cmux send-key-panel --panel surface:2 enter
```

## Read terminal output

```sh
cmux read-screen                                           # visible viewport
cmux read-screen --scrollback                              # include scrollback
cmux read-screen --lines 200                               # last 200 lines
cmux read-screen --surface surface:2 --scrollback --lines 200
cmux capture-pane --scrollback --lines 100                 # tmux alias
```

## Pipe output

```sh
cmux pipe-pane --command "grep ERROR"                      # pipe to shell command
cmux pipe-pane --command "tee output.log" --surface surface:2
```
