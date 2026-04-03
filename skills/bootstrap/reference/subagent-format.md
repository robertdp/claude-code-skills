# Custom Subagent Format Reference

Complete technical reference for Claude Code custom subagent definitions (`.claude/agents/`).

## File Format

Markdown files with YAML frontmatter. The markdown body is the system prompt.

```markdown
---
name: agent-name
description: When to delegate to this agent
tools: Read, Glob, Grep
model: sonnet
---

System prompt content. This is what the subagent sees as its instructions.
```

## Frontmatter Fields

| Field | Required | Type | Default | Description |
|-------|----------|------|---------|-------------|
| `name` | Yes | string | — | Unique identifier. Lowercase letters and hyphens. |
| `description` | Yes | string | — | When Claude should delegate. Critical for delegation accuracy. |
| `tools` | No | string or list | Inherits all | Tool allowlist. Agent can ONLY use these. |
| `disallowedTools` | No | string or list | None | Tools to deny. Applied first when both `tools` and `disallowedTools` are set; `tools` resolves against the remaining pool. |
| `model` | No | `sonnet`\|`opus`\|`haiku`\|full ID\|`inherit` | `inherit` | Model alias, full model ID (e.g. `claude-opus-4-6`), or `inherit`. Resolution order: `CLAUDE_CODE_SUBAGENT_MODEL` env var > per-invocation model > frontmatter > parent. |
| `permissionMode` | No | string | `default` | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`. `auto` uses an AI classifier to evaluate each tool call. If parent uses `bypassPermissions`, it takes precedence. If parent uses `auto`, subagent inherits it and frontmatter is ignored. |
| `maxTurns` | No | integer | None | Max agentic turns. |
| `skills` | No | string or list | None | Skills to preload into context. Full content injected. |
| `mcpServers` | No | object or list | None | MCP servers available. Each entry is either an inline definition (server name as key, full MCP config as value) or a string referencing an already-configured server. Inline servers connect on start, disconnect on finish. |
| `hooks` | No | object | None | Lifecycle hooks scoped to this subagent. |
| `memory` | No | `user`\|`project`\|`local` | None | Persistent memory scope. |
| `background` | No | boolean | `false` | Always run as background task. |
| `effort` | No | `low`\|`medium`\|`high`\|`max` | `inherit` | Effort level override. `max` is Opus 4.6 only. |
| `isolation` | No | `worktree` | None | Run in temporary git worktree. |
| `color` | No | string | None | UI display colour: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`. |
| `initialPrompt` | No | string | None | Auto-submitted as first user turn when agent runs as main session (`--agent` or `agent` setting). Commands and skills are processed. Prepended to any user-provided prompt. |

## File Locations and Priority

| Location | Scope | Priority |
|----------|-------|----------|
| Managed settings | Organization-wide | 1 (highest) |
| `--agents` CLI flag | Current session only | 2 |
| `.claude/agents/` | Current project | 3 |
| `~/.claude/agents/` | All projects | 4 |
| Plugin `agents/` directory | Where plugin enabled | 5 (lowest) |

When multiple subagents share the same name, the higher-priority location wins.

## Built-in Agents

| Agent | Model | Tools | Purpose |
|-------|-------|-------|---------|
| `Explore` | Haiku | Read-only | Fast codebase exploration. Thoroughness: quick, medium, very thorough. |
| `Plan` | Inherits | Read-only | Research during plan mode. |
| `general-purpose` | Inherits | All tools | Complex multi-step tasks. |
| `Bash` | Inherits | Terminal | Running terminal commands in a separate context. |
| `statusline-setup` | Sonnet | Read, Edit | Configure the user's status line setting. |
| `Claude Code Guide` | Haiku | Read-only + WebSearch/WebFetch | Answer questions about Claude Code features. |

## Spawning Control

When an agent runs as main thread (`claude --agent`), restrict spawnable subagents via `Agent()`:

```yaml
tools: Agent(worker, researcher), Read, Bash
```

- `Agent(worker, researcher)` — only these two can be spawned
- `Agent` (no parens) — any subagent can be spawned
- `Agent` omitted — no subagents at all

> In v2.1.63, the Task tool was renamed to Agent. Existing `Task(...)` references still work as aliases.

**Subagents cannot spawn other subagents** (no nesting).


## Persistent Memory

```yaml
memory: user
```

| Scope | Location | Use When |
|-------|----------|----------|
| `user` | `~/.claude/agent-memory/<name>/` | Cross-project learnings |
| `project` | `.claude/agent-memory/<name>/` | Project-specific, version controllable |
| `local` | `.claude/agent-memory-local/<name>/` | Project-specific, not committed |

When enabled: system prompt includes memory instructions, first 200 lines or 25KB of `MEMORY.md` (whichever comes first) auto-included, Read/Write/Edit auto-enabled.

## Skills Preloading

```yaml
skills:
  - api-conventions
  - error-handling-patterns
```

Full skill content injected at startup. Subagents do NOT inherit parent skills — list explicitly.

## Hooks

```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh $TOOL_INPUT"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/lint.sh"
```

Frontmatter events: `PreToolUse`, `PostToolUse`, `Stop` (→ `SubagentStop` at runtime).
Project-level events: `SubagentStart`, `SubagentStop` (in settings.json).

## Foreground vs Background

**Foreground** (default): Blocks main conversation. Permission prompts pass through. AskUserQuestion works.

**Background** (`background: true`): Runs concurrently. Permissions pre-approved at launch. AskUserQuestion fails (agent continues).

Background agents cannot prompt for permissions. For write work in background, use a custom subagent with `permissionMode: acceptEdits`. See [decision-frameworks.md](decision-frameworks.md) for the full constraint table.

## How Results Return

- Subagent runs in its own context window
- Returns a summary to the main conversation (not full transcript)
- Auto-compaction at ~95% capacity

## Context Isolation

Custom subagents start with a clean slate — no conversation history, only system prompt + preloaded skills + CLAUDE.md. Pass all needed context explicitly in the Agent prompt. See [decision-frameworks.md](decision-frameworks.md) for when to isolate vs share context.

## Template

```markdown
---
name: focused-worker
description: Performs [task]. Delegate when [condition].
tools: Read, Glob, Grep
model: sonnet
maxTurns: 10
---

You are a [role]. Your sole objective is to [specific task].

## What you receive
- [Input]: description

## What you produce
- [Output]: structured summary

## Constraints
- Only examine [scope]
- Do not [anti-pattern]
- Report with file:line references

## Process
1. [Step]
2. [Step]
3. [Step]
```

## System Prompt Best Practices

- Single objective, first sentence
- Define inputs and outputs explicitly
- Set constraints and non-goals
- One agent, one job
- Specify output format for reliable consumption by main agent
- Include process steps for complex tasks
- Explicitly instruct against `run_in_background: true` on Agent/Bash calls — returned output is lost (the caller never sees it). File output still works, but if the caller needs to act on the result, the call must run in the foreground.
