# Plugin Format Reference

Complete technical reference for Claude Code plugins.

## Overview

A plugin is a self-contained directory extending Claude Code with skills, agents, hooks, MCP servers, and LSP servers. Plugins are namespaced (`plugin-name:component`) to prevent conflicts.

## When Plugin vs Standalone

| Approach | Skill names | Best for |
|----------|-------------|----------|
| **Standalone** (`.claude/`) | `/hello` | Personal workflows, project-specific, quick experiments |
| **Plugin** (`.claude-plugin/plugin.json`) | `/plugin-name:hello` | Sharing with team/community, versioned releases, reusable across projects |

Use standalone first, convert to plugin when ready to share.

## Directory Structure

```
my-plugin/
├── .claude-plugin/           # Manifest only — nothing else here
│   └── plugin.json
├── skills/                   # Skills with SKILL.md
│   └── code-review/
│       ├── SKILL.md
│       └── reference.md
├── commands/                 # Legacy skill location (use skills/ for new)
│   └── status.md
├── agents/                   # Subagent definitions
│   └── reviewer.md
├── hooks/                    # Hook configurations
│   └── hooks.json
├── .mcp.json                 # MCP server definitions
├── .lsp.json                 # LSP server configurations
├── settings.json             # Default settings (only `agent` key supported)
├── scripts/                  # Hook and utility scripts
│   └── validate.sh
└── README.md
```

**Critical**: Components go at plugin root, NOT inside `.claude-plugin/`. Only `plugin.json` goes in `.claude-plugin/`.

## Plugin Manifest Schema

File: `.claude-plugin/plugin.json`. Optional — if omitted, name derived from directory, components auto-discovered.

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique identifier. Kebab-case. Used as namespace prefix. |

### Metadata Fields

| Field | Type | Description |
|-------|------|-------------|
| `version` | string | Semantic version (`MAJOR.MINOR.PATCH`) |
| `description` | string | Brief plugin purpose |
| `author` | object | `{name, email?, url?}` |
| `homepage` | string | Documentation URL |
| `repository` | string | Source code URL |
| `license` | string | SPDX identifier (`MIT`, `Apache-2.0`) |
| `keywords` | array | Discovery tags |

### Component Path Fields

Custom paths supplement defaults — they don't replace them. All paths relative to plugin root, starting with `./`.

| Field | Type | Description |
|-------|------|-------------|
| `commands` | string\|array | Additional command files/directories |
| `agents` | string\|array | Additional agent files |
| `skills` | string\|array | Additional skill directories |
| `hooks` | string\|array\|object | Hook config paths or inline config |
| `mcpServers` | string\|array\|object | MCP config paths or inline config |
| `lspServers` | string\|array\|object | LSP config paths or inline config |
| `outputStyles` | string\|array | Additional output style files/directories. Custom output format configurations. |

### Complete Example

```json
{
  "name": "enterprise-tools",
  "version": "2.1.0",
  "description": "Enterprise workflow automation",
  "author": { "name": "DevTools Team" },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/company/plugin",
  "license": "MIT",
  "keywords": ["enterprise", "automation"],
  "commands": ["./custom/commands/"],
  "agents": ["./custom/agents/reviewer.md"],
  "skills": "./custom/skills/",
  "hooks": "./config/hooks.json",
  "mcpServers": "./mcp-config.json",
  "lspServers": "./.lsp.json"
}
```

## Environment Variables

`${CLAUDE_PLUGIN_ROOT}` — absolute path to plugin directory. Use in hooks, MCP configs, and scripts.

```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
}
```

## Component Specifications

### Skills

Location: `skills/<name>/SKILL.md` (or `commands/<name>.md` for legacy).

Same format as standalone skills. Namespaced as `plugin-name:skill-name`. See [skill-format.md](skill-format.md).

### Agents

Location: `agents/<name>.md`.

Same format as standalone agents. Namespaced as `plugin-name:agent-name`. See [subagent-format.md](subagent-format.md).

### Hooks

Location: `hooks/hooks.json` or inline in `plugin.json`.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"
          }
        ]
      }
    ]
  }
}
```

Available events: `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `Notification`, `SessionStart`, `SessionEnd`, `Stop`, `SubagentStart`, `SubagentStop`, `PreCompact`, `PermissionRequest`, `Setup`, `TeammateIdle`, `TaskCompleted`, `ConfigChange`, `UserPromptSubmit`, `WorktreeCreate`, `WorktreeRemove`.

Hook types: `command` (shell), `http` (POST to URL), `prompt` (single-turn LLM evaluation), `agent` (subagent verifier with tool access).

### MCP Servers

Location: `.mcp.json` at plugin root, or inline in `plugin.json`.

```json
{
  "mcpServers": {
    "plugin-db": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": { "DB_PATH": "${CLAUDE_PLUGIN_ROOT}/data" }
    }
  }
}
```

Servers start automatically when plugin is enabled.

### LSP Servers

Location: `.lsp.json` at plugin root, or inline in `plugin.json`.

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": { ".go": "go" }
  }
}
```

Required fields: `command`, `extensionToLanguage`.

Optional: `args`, `transport` (`stdio`|`socket`), `env`, `initializationOptions`, `settings`, `workspaceFolder`, `startupTimeout`, `shutdownTimeout`, `restartOnCrash`, `maxRestarts`.

Users must install the language server binary separately.

### Settings

Location: `settings.json` at plugin root. Only `agent` key currently supported.

```json
{ "agent": "security-reviewer" }
```

Activates a plugin agent as the main thread. Settings from `settings.json` override `plugin.json` settings.

## Constraints

- Installed plugins cannot reference files outside their directory (use symlinks)
- `settings.json` only supports the `agent` key currently
- All component paths must be relative, starting with `./`
- **Plugin subagent restrictions**: plugin agents do NOT support `hooks`, `mcpServers`, or `permissionMode` frontmatter fields — these are silently ignored when loaded from a plugin. To use them, copy the agent file into `.claude/agents/` or `~/.claude/agents/`.
