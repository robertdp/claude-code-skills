# Sidebar & Notifications

## Status pills

Key-value pairs shown in the sidebar. Use unique keys per tool/concern.

```sh
cmux set-status build "compiling" --icon hammer --color "#ff9500"
cmux set-status deploy "v1.2.3"
cmux set-status claude "thinking" --icon sparkle --color "#7c3aed"
cmux clear-status build
cmux list-status
cmux list-status --workspace workspace:2
```

## Progress bar

Value from 0.0 to 1.0.

```sh
cmux set-progress 0.0 --label "Starting..."
cmux set-progress 0.5 --label "Building..."
cmux set-progress 1.0 --label "Done"
cmux clear-progress
```

## Log entries

Levels: `info`, `progress`, `success`, `warning`, `error`

```sh
cmux log "Build started"
cmux log --level progress "Compiling module 3/10"
cmux log --level success -- "All 42 tests passed"
cmux log --level error --source build "Compilation failed on line 42"
cmux log --level warning --source lint "3 warnings found"
cmux clear-log
cmux list-log
cmux list-log --limit 20
```

## Sidebar state dump

Returns all metadata: cwd, git branch, ports, status, progress, logs.

```sh
cmux sidebar-state
cmux sidebar-state --workspace workspace:2
```

## Notifications

```sh
cmux notify --title "Build done" --body "All tests passed"
cmux notify --title "Error" --subtitle "test.swift" --body "Line 42: syntax error"
cmux list-notifications
cmux clear-notifications
```
