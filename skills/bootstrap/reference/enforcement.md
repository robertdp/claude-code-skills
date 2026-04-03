# Objective Enforcement

Mechanisms for keeping skills and agents focused on their stated objectives. Assessed case-by-case based on task fragility.

## Assessment Framework

Map task fragility to enforcement level:

| Freedom Level | Task Type | Enforcement | Examples |
|---------------|-----------|-------------|----------|
| High | Creative, analytical | Prompt-level only | Code review, content generation, research, analysis |
| Medium | Standard operations | Prompt + tool restrictions | File processing, API calls, data transforms, formatting |
| Low | Fragile, dangerous | Prompt + tools + hooks | DB migrations, payments, security ops, deployments |

**Rule**: Start with the minimum enforcement that produces reliable results. Add layers only when a less-restricted version demonstrably drifts.

## Prompt-Level Enforcement

The foundation. Every skill and agent needs at minimum:

### Objective tag
State the single purpose clearly. One sentence is ideal.
```xml
<objective>
Format SQL queries consistently following the project's style guide.
</objective>
```

### Success criteria
Verifiable conditions that define "done." The agent checks these before stopping.
```xml
<success_criteria>
- All queries formatted with UPPERCASE keywords
- Indentation consistent at 2 spaces per level
- No semantic changes to any query
</success_criteria>
```

### Scope boundaries
What's in and what's out. Explicit non-goals prevent drift into adjacent concerns.
```xml
<objective>
Review pull requests for correctness and security issues.

Non-goals:
- Style or formatting feedback (handled by linter)
- Performance optimization suggestions (separate review)
- Refactoring recommendations (out of scope)
</objective>
```

### Anti-drift instructions
For tasks where drift is a known risk:
```
Focus exclusively on [stated objective]. If you encounter issues outside this scope,
note them briefly but do not investigate or fix them. Return to the primary task.
```

## Permission Modes (Subagents)

The `permissionMode` field on custom subagents explicitly grants permissions scoped to that agent:

| Mode | Effect | Use When |
|------|--------|----------|
| `default` | Normal permission checks | Standard foreground work |
| `acceptEdits` | Auto-approve Write/Edit/NotebookEdit | Background agents that write files |
| `auto` | AI classifier evaluates each tool call | Balanced automation with safety checks |
| `dontAsk` | Deny instead of prompting | Strict read-only enforcement |
| `bypassPermissions` | Auto-approve everything | Fully trusted automation |
| `plan` | Plan mode only | Research and planning agents |

**Critical for background agents**: Background agents cannot prompt for permissions. An ad-hoc Agent call that needs Write/Edit will fail silently in background. Custom subagents with `permissionMode: acceptEdits` solve this — the permission is explicit, scoped, and auditable.

Always grant the **minimum necessary mode**. Prefer `acceptEdits` over `bypassPermissions` when only file writes are needed.

## Tool Restrictions

The `allowed-tools` field (skills) and `tools` field (subagents) serve dual purposes:
1. **Permit**: Auto-approve these tools without user confirmation
2. **Restrict**: Agent can ONLY use these tools — everything else is blocked

### Patterns by freedom level

**High freedom** (creative):
```yaml
# No tool restriction — let the agent use what it needs
# Or broad access:
allowed-tools: Read, Write, Edit, Grep, Glob, Agent
```

**Medium freedom** (standard ops):
```yaml
# Read + targeted write
allowed-tools: Read, Grep, Glob, Edit
```

**Low freedom** (fragile):
```yaml
# Read-only
allowed-tools: Read, Grep, Glob

# Or specific commands only
allowed-tools: Read, Bash(python scripts/migrate.py *)
```

### Bash restrictions
Bash supports glob-style command restrictions:
```yaml
allowed-tools: Bash(git *)           # Only git commands
allowed-tools: Bash(npm test *)      # Only npm test
allowed-tools: Bash(python scripts/*) # Only scripts in scripts/
```

## Lifecycle Hooks

Runtime validation that runs automatically during skill/agent execution.

### Hook Events

| Event | Matcher input | When it fires |
|-------|--------------|---------------|
| `PreToolUse` | Tool name | Before a tool executes. Can block it. |
| `PostToolUse` | Tool name | After a tool succeeds. |
| `PostToolUseFailure` | Tool name | After a tool fails. |
| `PermissionRequest` | Tool name | When a permission dialog appears. |
| `Stop` | — | When Claude finishes responding. |
| `SubagentStart` | Agent type | When a subagent is spawned. |
| `SubagentStop` | Agent type | When a subagent finishes. |
| `TeammateIdle` | — | When an agent team teammate is about to go idle. |
| `TaskCreated` | — | When a task is being created. |
| `TaskCompleted` | — | When a task is being marked complete. |
| `SessionStart` | Session type | When a session begins or resumes. |
| `SessionEnd` | End reason | When a session terminates. |
| `UserPromptSubmit` | — | When user submits a prompt, before Claude processes it. |
| `Notification` | Notification type | When Claude Code sends a notification. |
| `PreCompact` | Trigger type | Before context compaction. |
| `ConfigChange` | Config source | When a configuration file changes. |
| `Setup` | — | When a plugin or component is first set up. |
| `WorktreeCreate` | — | When a worktree is being created. |
| `WorktreeRemove` | — | When a worktree is being removed. |

Events without matcher support (—) always fire on every occurrence.

### Common hook examples

**PreToolUse** — validate commands before execution, check file paths, enforce naming:
```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
```

**PostToolUse** — lint after edits, validate output format, log changes:
```yaml
hooks:
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
```

**Stop** — final validation, cleanup, summary reports:
```yaml
hooks:
  Stop:
    - hooks:
        - type: command
          command: "./scripts/final-validation.sh"
```

### Hook handler types

| Type | Description |
|------|-------------|
| `command` | Run a shell command. Receives event JSON on stdin. |
| `http` | POST event JSON to a URL endpoint. |
| `prompt` | Single-turn LLM evaluation. Returns yes/no decision. |
| `agent` | Spawn a subagent verifier with tool access to check conditions. |

### Hook handler fields

**Common fields** (all types):

| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | `command`, `http`, `prompt`, or `agent` |
| `timeout` | No | Seconds before cancelling. Defaults: 600 (command), 30 (prompt/http), 60 (agent) |
| `statusMessage` | No | Custom spinner message while hook runs |
| `once` | No | If `true`, runs only once per invocation then removed. Skills only. |

**Command-specific**: `command` (shell string), `async` (boolean — run without blocking)

**HTTP-specific**: `url` (POST target), `headers` (key-value), `allowedEnvVars` (env vars permitted in header interpolation)

**Prompt-specific**: `prompt` (text sent to Claude for evaluation)

### Matcher
`matcher` is a regex filtering the event-specific field: tool names for tool events (`"Bash"`, `"Edit|Write"`, `"mcp__.*"`), agent type for subagent events, etc. Omit or use `"*"` to match all occurrences.

## disable-model-invocation

```yaml
disable-model-invocation: true
```

Removes the skill from Claude's auto-invocation context entirely. Only the user can trigger it via `/skill-name`.

Use for:
- Deployment skills
- Data migration skills
- Skills that send external messages (Slack, email, GitHub comments)
- Any skill with irreversible side effects

## Anti-Drift Techniques

### Clear scope statements
State what the skill does AND what it doesn't do. "Non-goals" sections are more effective than general warnings.

### Success criteria as checklist
Make criteria specific and verifiable. The agent should be able to check each one before declaring completion.

### Periodic re-grounding
For long-running workflows, include re-grounding steps:
```xml
<step name="checkpoint">
Before proceeding, re-read the objective and verify:
- Current work aligns with the stated goal
- No scope expansion has occurred
- All decisions trace back to a user requirement
</step>
```

### Output validation
After producing output, validate against the original requirements — not against what "seems good." The critical review subagent pattern is effective here: a separate agent with clean context reviews the output against the original brief.

### Explicit handoff criteria
Define when to stop and hand back to the user. Prevents the agent from continuing to "improve" beyond what was asked.

## Compound Enforcement

Combine mechanisms for high-risk tasks:

```yaml
---
name: db-migrate
description: Run database migrations. Use when user says "migrate" or "run migrations".
disable-model-invocation: true
allowed-tools: Read, Bash(python scripts/migrate.py *)
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-migration.sh $TOOL_INPUT"
          once: false
  PostToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/check-migration-status.sh"
---
```

This combines:
1. `disable-model-invocation` — user must explicitly invoke
2. `allowed-tools` — only read + specific migration script
3. `PreToolUse` hook — validate every command before execution
4. `PostToolUse` hook — verify state after each step

