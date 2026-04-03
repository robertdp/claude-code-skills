# Decision Frameworks

When to use what. These frameworks drive architectural choices across all bootstrap workflows.

## Single-Agent Default

Multi-agent coordination is not a neutral choice — it frequently degrades performance. Start with a single agent. Escalate to multi-agent only when all three conditions hold:

1. **The task is genuinely decomposable.** Independent subtasks that don't require sequential handoff. High sequential dependency degrades under coordination even when complexity is high.
2. **A single agent can't handle it adequately.** If one agent already produces good results, adding coordination yields diminishing or negative returns.
3. **The coordination value exceeds the overhead.** Coordination cost scales super-linearly with agent count. Each additional agent adds more overhead than the last.

When evaluating a multi-agent design, also consider: tool-heavy tasks (many tool calls per step) suffer disproportionately from multi-agent overhead. The more tools involved, the higher the bar for justifying coordination.

## Inline vs Subagent

**Keep work inline** when:
- Frequent back-and-forth with the user is needed
- Iterative refinement where each step depends on the last
- Shared context from earlier conversation is essential
- Latency-sensitive — subagent spawn has overhead
- The task is simple enough that delegation adds complexity without value

**Delegate to a subagent (Agent tool)** when:
- Output is verbose and would pollute main context (test suites, log analysis, large codebases)
- Tool restrictions are needed — subagent enforces a narrower tool set
- Work is self-contained and returns a summary
- Parallel research — multiple subagents can search simultaneously
- Context isolation benefits the task — clean slate with no accumulated noise
- The main conversation has drifted and you need a fresh perspective on the problem

**Rule of thumb**: If the result matters more than the journey, delegate. If the conversation matters, keep inline.

## Custom Subagent vs Ad-Hoc Agent

**Define a custom subagent** (`.claude/agents/`) when:
- The agent will be reused across multiple sessions or skills
- Specific tool restrictions must be enforced consistently
- Persistent memory is needed (learning across invocations)
- Skills should be preloaded into the agent's context
- Lifecycle hooks are needed for validation or logging
- The agent benefits from context isolation with a well-defined system prompt
- The description needs to be precise for accurate delegation by the main agent
- **The agent needs explicit permissions** — `permissionMode` grants permissions scoped to that agent

**Use an ad-hoc Agent call** when:
- One-off research or exploration
- Simple delegation with a prose prompt
- No special tool configuration needed
- The task is specific to this conversation and won't recur
- You need to pass conversation-specific context that doesn't fit a reusable template

**Key insight**: Custom subagents are like reusable functions. Ad-hoc Agent calls are like inline code. Extract to a custom subagent when you find yourself writing the same Agent prompt repeatedly.

### Permissions and Background Execution

Ad-hoc Agent calls inherit the parent's permission context. They cannot be granted extra permissions. **Background agents cannot prompt for permissions** — if a tool isn't pre-approved, the call fails silently.

This creates a hard constraint:

| Execution | Needs Write/Edit | Solution |
|-----------|-----------------|----------|
| Foreground ad-hoc Agent | Yes | Works — can prompt the user |
| Background ad-hoc Agent | Yes | **Fails** — cannot prompt |
| Background ad-hoc Agent | No (read-only) | Works — Read/Grep/Glob don't need approval |
| Custom subagent (any) | Yes | Works — set `permissionMode: acceptEdits` or `auto` |

**Rule**: If a subagent needs to write files and may run in the background, it **must** be a custom subagent with `permissionMode` set. Ad-hoc Agent calls are only safe for background read-only work.

### Model Resolution Order

When a subagent is invoked, the model is resolved in this order (first match wins):

1. `CLAUDE_CODE_SUBAGENT_MODEL` environment variable
2. Per-invocation `model` parameter on the Agent call
3. Subagent definition's `model` frontmatter
4. Main conversation's model (inherit)

### Parallel Execution vs Background Mode

`run_in_background` and parallel execution are **different mechanisms**:

- **Parallel execution**: Multiple foreground Agent calls in a single message run concurrently. Results return to the main conversation when all complete.
- **Background mode** (`run_in_background=true`): A single agent runs while the main conversation continues with other work. Results must be retrieved later via `TaskOutput`.

**`run_in_background=true` silently discards conversational return values.** The agent completes successfully but the output returned to the caller is empty (0 bytes). No error is surfaced — silent data loss. Tool calls within the agent (Write, Bash, etc.) still work and persist normally. The agent does its job; you just can't read the result through the normal Agent return channel.

**Two viable patterns:**

| Goal | Approach |
|------|----------|
| Run N specialists in parallel, read results inline | N foreground Agent calls in one message — results return normally |
| Run N specialists in parallel, read results from disk | N background Agent calls — each agent Writes output to a designated file path, caller reads files after completion |

## Skill vs Subagent

| Dimension | Skill | Custom Subagent |
|-----------|-------|-----------------|
| Invocation | User types `/name` or Claude auto-invokes | Claude delegates via Agent tool |
| Runs in | Main conversation context (or forked) | Isolated context window |
| User interaction | Full (AskUserQuestion, etc.) | Limited (foreground only) |
| Reusability | High — across sessions and projects | High — across sessions |
| Tool control | `allowed-tools` in frontmatter | `tools` / `disallowedTools` in frontmatter |
| Custom tools | No | No |
| Best for | Interactive workflows, multi-step processes | Focused background work, enforced constraints |

**Build a skill** when the workflow is interactive, reusable, and benefits from user involvement.

**Build a subagent** when the work is focused, self-contained, and benefits from isolation or tool restriction.

## Standalone vs Plugin

**Use standalone configuration** (`.claude/`) when:
- Personal workflow for a single project
- Quick experiments before packaging
- Short skill names preferred (`/hello` vs `/plugin-name:hello`)
- No need to share or distribute

**Package as a plugin** when:
- Sharing with team or community
- Same skills/agents needed across multiple projects
- Benefits from versioned releases and updates
- Has multiple components that belong together (skills + hooks + agents)
- Distributing through a marketplace

**Migration path**: Start standalone, convert to plugin when ready to share. Copy `skills/`, `agents/`, `commands/` to plugin root. Move hooks from `settings.json` to `hooks/hooks.json`.

**Key insight**: A plugin is a distribution mechanism, not a different architecture. The skills and agents inside a plugin follow the same format as standalone ones. The plugin adds namespacing, versioning, and packaging.

## Subagents vs Agent Teams

**Use subagents** when:
- Tasks are independent and don't need to communicate with each other
- Only the result matters, not the discussion
- Cost efficiency is important
- The main agent can effectively coordinate all work
- Tasks are focused and well-defined

**Use agent teams** when:
- Workers need to share findings and challenge each other's conclusions
- Cross-layer coordination is required (frontend + backend + tests)
- Debugging with competing hypotheses benefits from multiple perspectives
- Research requires genuinely different viewpoints
- Inter-agent communication adds value that justifies the cost

**Cost check**: Teams cost 2-5x per teammate. Before using teams, ask: "Does the coordination value exceed the cost of running N independent Claude instances?" If the answer is "the tasks are independent and just need results," subagents are cheaper and simpler.

## Context Isolation

Custom subagents start with a **clean slate** — no conversation history, no accumulated noise, no stale assumptions. Only the system prompt + preloaded skills + CLAUDE.md.

**Benefits of clean-slate context**:
- Lower error rate on focused tasks (no irrelevant context to misinterpret)
- Precise input: the agent sees exactly what you pass, nothing more
- No drift from prior conversation tangents
- Reproducible: same input → same behaviour regardless of conversation state

**Costs of isolation**:
- No shared context — everything needed must be passed explicitly
- Higher latency for context-dependent chains (each agent starts cold)
- State passing overhead — you must design input/output contracts

**Use isolation when**:
- The task is self-contained and doesn't need conversation history
- Main context has accumulated irrelevant information
- You want predictable, reproducible behaviour
- The work is long-running and benefits from a fresh context window

**Keep shared context when**:
- The task requires knowledge of what was just discussed
- Multiple rapid exchanges are needed
- The overhead of explicit state passing exceeds the benefit of isolation

## Model Selection

| Model | Cost | Speed | Best For |
|-------|------|-------|----------|
| Haiku | Low | Fast | Read-only exploration, simple analysis, high-volume parallel tasks |
| Sonnet | Medium | Medium | Balanced work, code review, standard implementation |
| Opus | High | Slower | Complex reasoning, architectural decisions, nuanced judgment |

**Defaults**:
- Exploration subagents → Haiku (fast, cheap, read-only is sufficient)
- Implementation subagents → Sonnet (good balance of capability and cost)
- Critical review subagents → Sonnet or Opus (need judgment)
- Main orchestrator → Opus (coordinates everything, makes key decisions)

**Rule**: Use the cheapest model that produces reliable results for the task. Promote to a more capable model only when cheaper models produce errors or miss nuance.
