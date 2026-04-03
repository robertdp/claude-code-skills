---
name: bootstrap
description: Build Claude Code skills, custom subagents, plugins, and agent team configurations from requirements. Handles orchestration design, objective enforcement, and self-updating reference material. Use when user says "build a skill", "create an agent", "build a plugin", "design orchestration", "bootstrap", or invokes /bootstrap or /robert:bootstrap.
argument-hint: "[workflow] [description]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(ls *), Agent, WebSearch, WebFetch
---

<objective>
Meta-skill: an LLM generating instructions for LLMs. Builds skills and agents that stay focused on their objectives through appropriate structure, tool restrictions, and enforcement mechanisms.

Produces:
- Skills (SKILL.md + reference/workflow files)
- Custom subagents (.claude/agents/ definitions)
- Plugins (.claude-plugin/ packages with skills, agents, hooks, MCP, LSP)
- Agent team configurations
- Orchestration topologies for multi-agent systems
</objective>

<essential_principles>
These apply to ALL workflows:

1. **Understand before building.** Interview first. Gather requirements through structured questioning. Only ask what you can't learn from the codebase or web. For simple, well-specified requests, skip the interview and proceed directly.

2. **Solve the real problem.** Before designing structure, understand what expert-level output looks like in the domain. Research what practitioners actually evaluate — don't accept surface-level framing or default to easily-observable patterns. For every agent: if its work could be fully replicated by grep, regex, or existing tooling, the scope is too shallow. Push agents to exercise judgment about context, relationships, and trade-offs.

3. **Power over brevity.** Use tokens where they improve reliability. Cut only where it provably doesn't hurt. A skill that works consistently is worth more than a minimal one that improvises.

4. **Match enforcement to risk.** Assess task fragility. Creative tasks get high freedom with prompt-level guidance. Standard operations get tool restrictions. Fragile operations get hooks and exact instructions. See [reference/enforcement.md](reference/enforcement.md).

5. **Context isolation is a tool.** Custom subagents start with a clean slate — no accumulated noise, no stale assumptions. Use this when work benefits from focused, low-noise context rather than inheriting the full conversation. See [reference/decision-frameworks.md](reference/decision-frameworks.md).

6. **Progressive disclosure.** SKILL.md under 500 lines. Reference files for detailed docs. One level of nesting. Load only what the current workflow needs.

7. **Verify against reality.** Before embedding technical details in a skill, verify against current docs and the codebase. Training data goes stale; documented reality doesn't. Use WebSearch when uncertain.

8. **Write then review.** Produce the files, then walk through them with the user for iteration. Don't ask for approval of a design sketch — show the real thing.

9. **Critical review before handoff.** After writing files, launch a subagent (Agent, subagent_type=general-purpose) to critically review the output against the original requirements. The reviewer checks for meta-drift: scope creep, enforcement mismatch, instructions serving adjacent concerns rather than the stated objective. Present findings to the user alongside the walkthrough.

10. **No background tasks in produced output.** Skills and subagents must explicitly instruct against setting `run_in_background: true` on Agent or Bash calls. Background execution does not return output to the caller — the main agent never sees the result, so instructions like "respond with X when done" silently fail. Output can still be written to files, but if the caller needs to act on the result, the call must run in the foreground.

11. **Appropriate generality.** Some skills own their structure — a note-taking skill defines its folder layout, a deployment skill targets a specific service. These should embed the specifics they control. Other skills operate on structure they don't own — analysing codebases, reviewing documents, auditing dependencies. These should discover context at runtime (read indexes, Glob, Grep) rather than hardcode what was found during the build. Match the level of specificity to what the skill owns vs what it consumes. When uncertain, ask.

### Output Location

Ask the user where to place output:
- **Personal**: `~/.claude/skills/` or `~/.claude/agents/`
- **Project**: `.claude/skills/` or `.claude/agents/`

Default to personal if not in a git repository. Ask if ambiguous.
</essential_principles>

<interview>
Adapted from the interview skill. Run before routing to a workflow.

### Domain Detection

Detect from the user's description:
- **Skill authoring**: building a reusable skill with SKILL.md
- **Agent design**: creating a custom subagent
- **Plugin development**: packaging skills, agents, hooks, MCP/LSP into a distributable plugin
- **Orchestration**: designing how multiple agents work together
- **Team coordination**: setting up agent teams

### Question Strategy

- 2-4 questions per round via AskUserQuestion
- Progress from broad to specific
- Between rounds: explore codebase if relevant, WebSearch if domain is unfamiliar
- For analytical or audit skills: research what domain experts actually evaluate before designing agents. The user describes the problem — you must understand the full solution space.
- Populate options from actual findings, not generic defaults
- Stop when no new unknowns surface (typically 1-3 rounds)

### What to Extract

| Question | Why |
|----------|-----|
| What should it do? What triggers it? | Core purpose and description |
| Personal or project scope? | Output location |
| Single-purpose or multi-workflow? | Simple vs router pattern |
| Does it need subagents? | Orchestration design |
| What's the risk level of operations? | Enforcement assessment |
| Read-only or can it modify? | Tool restrictions |
| User-triggered or auto-invoked? | Visibility settings |
| Does it accept arguments? | argument-hint, $ARGUMENTS usage |
| What does expert-level output look like in this domain? What would a naive approach miss? | Depth calibration — ensures the skill targets substantive work |

### When to Skip

Skip the interview for well-specified requests where all the above are clear from the user's input. Proceed directly to routing.

### Backtracking

If an answer contradicts or invalidates an assumption from an earlier round — or conflicts with what you found in the codebase — revisit that question. State what changed and re-confirm before continuing.
</interview>

<intake>
<arguments>
$ARGUMENTS
</arguments>

Parse the arguments above for a workflow name (first word). Route to the matching workflow. Pass remaining arguments as context for that workflow.

If no workflow name is present or it's ambiguous, ask:

What would you like to build?

1. **Skill** — A reusable skill (SKILL.md + supporting files)
2. **Subagent** — A custom subagent definition (.claude/agents/)
3. **Plugin** — A distributable package (skills, agents, hooks, MCP, LSP)
4. **Team** — An agent team configuration
5. **Orchestration** — Design a multi-agent topology
6. **Self-update** — Research latest docs and update bootstrap's reference material

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, first arg is "skill" | [workflows/skill.md](workflows/skill.md) |
| 2, first arg is "subagent" or "agent" | [workflows/subagent.md](workflows/subagent.md) |
| 3, first arg is "plugin" | [workflows/plugin.md](workflows/plugin.md) |
| 4, first arg is "team" | [workflows/team.md](workflows/team.md) |
| 5, first arg is "orchestration" or "topology" | [workflows/orchestration.md](workflows/orchestration.md) |
| 6, first arg is "self-update" or "update" | [workflows/self-update.md](workflows/self-update.md) |

After reading the workflow, follow it exactly. The interview section above should be completed before or during the workflow's first step — use judgment based on how much information the user provided upfront.
</routing>
