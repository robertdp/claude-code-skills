# Agent Teams Reference

Claude Code Agent Teams specification. Experimental feature.

Experimental: requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in settings.json env block. Requires Claude Code v2.1.32+.

## Architecture

| Component | Role |
|-----------|------|
| Team lead | Main session. Creates team, spawns teammates, coordinates. |
| Teammates | Separate Claude Code instances with own context windows. |
| Task list | Shared work items. File-locking prevents race conditions. |
| Mailbox | Inter-agent messaging. |

## Differences from Subagents

| | Subagents | Agent Teams |
|--|-----------|-------------|
| Context | Own window; results return to caller | Own window; fully independent |
| Communication | Report back to main only | Message each other directly |
| Coordination | Main agent manages | Shared task list, self-coordination |
| Best for | Focused tasks, result-only | Complex work requiring discussion |
| Cost | Lower: results summarised | Higher: ~2-5x per teammate |

## Task Management

States: **pending** → **in progress** → **completed**.

Tasks can have dependencies — pending tasks with unresolved deps cannot be claimed. File locking prevents race conditions on claims.

## Communication

- **message**: Send to one specific teammate
- **broadcast**: Send to all (use sparingly — costs scale with team size)
- Automatic message delivery and idle notifications
- Teammates load same project context (CLAUDE.md, MCP, skills) but NOT lead's conversation history

## Quality Gates

### TeammateIdle
Runs when teammate is about to go idle.
- Exit 0: goes idle
- Exit 2: sends feedback, keeps working

### TaskCreated
Runs when a task is being created.
- Exit 0: task is created
- Exit 2: prevents creation, sends feedback

### TaskCompleted
Runs when task being marked complete.
- Exit 0: task completes
- Exit 2: prevents completion, sends feedback

## Plan Approval

Lead can require teammates to plan before implementing:

```
Spawn an architect teammate to refactor the auth module.
Require plan approval before they make any changes.
```

Teammate works in read-only plan mode → submits plan → lead reviews → approves or rejects with feedback. Rejected teammates revise and resubmit. Lead makes approval decisions autonomously — influence via criteria in the prompt.

## Display Modes

| Mode | Setting | Behaviour |
|------|---------|-----------|
| `in-process` | Default | All teammates in main terminal. |
| `tmux` / `auto` | `teammateMode: "tmux"` or `"auto"` in settings.json | Split panes via tmux or iTerm2. Auto-detects. |

Set via `teammateMode` in `~/.claude.json` (global config). Default is `"auto"` (split panes if already in tmux, otherwise in-process). Override per-session with `--teammate-mode in-process`. Split panes not supported in VS Code terminal, Windows Terminal, or Ghostty.

## Storage

- **Team config**: `~/.claude/teams/{team-name}/config.json` (runtime state — don't edit by hand)
- **Task list**: `~/.claude/tasks/{team-name}/`

## Best Practices

- **Team size**: 3-5 teammates for most workflows. Beyond that, coordination overhead outweighs gains.
- **Task sizing**: 5-6 tasks per teammate keeps everyone productive. Too small → coordination exceeds benefit. Too large → risk of wasted effort without check-ins.
- **Start read-only**: Begin with research/review tasks. Scale to parallel implementation as you learn the coordination patterns.
- **Avoid file conflicts**: Break work so each teammate owns different files. Two teammates editing the same file causes overwrites.
- **Give teammates enough context**: They load project context (CLAUDE.md, MCP, skills) but NOT the lead's conversation history. Include task-specific details in spawn prompts.

## Limitations

- No session resumption for in-process teammates — `/resume` does not restore them
- Task status can lag — teammates sometimes fail to mark tasks completed, blocking dependents
- One team per session — clean up before starting a new one
- No nested teams — teammates cannot spawn their own teams
- Lead is fixed — cannot promote a teammate or transfer leadership
- Permissions set at spawn for all teammates — can change individually after, but not at spawn time
- Shutdown can be slow — teammates finish current request before stopping

## Cost

~2-5x tokens per teammate. Reserve for work that genuinely benefits from parallel perspectives.
