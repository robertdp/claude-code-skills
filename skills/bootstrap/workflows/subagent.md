<required_reading>
Read before starting:
- [reference/subagent-format.md](../reference/subagent-format.md) — Complete subagent spec
- [reference/enforcement.md](../reference/enforcement.md) — Enforcement mechanisms
- [reference/decision-frameworks.md](../reference/decision-frameworks.md) — When custom vs ad-hoc
</required_reading>

<process>

## Build a Custom Subagent

<step name="interview">
If the interview hasn't been completed, gather requirements now.

Extract via AskUserQuestion (2-4 questions per round):

**Purpose**:
- What does this agent do? (single sentence)
- When should Claude delegate to it? (delegation trigger)
- What does it receive as input? What does it return?

**Configuration**:
- Which tools does it need? (read-only, write, bash, specific commands)
- Which model? (Haiku for fast read-only, Sonnet for balanced, Opus for complex reasoning)
- Should it run in foreground or background?
- Does it need persistent memory? (cross-session learning)
- Should it preload any skills?
- Does it need git worktree isolation?

**Placement**:
- Personal (~/.claude/agents/) or project (.claude/agents/)?

Between rounds: explore the codebase for existing patterns, tools, or conventions the agent should follow.
</step>

<step name="justify">
Before building, justify why a custom subagent is the right choice over an ad-hoc Agent call:

**Custom subagent is right when** (check at least one):
- [ ] Will be reused across multiple sessions or skills
- [ ] Needs specific tool restrictions enforced consistently
- [ ] Benefits from persistent memory
- [ ] Needs skills preloaded
- [ ] Needs lifecycle hooks
- [ ] Benefits from context isolation with a well-defined system prompt
- [ ] Description must be precise for accurate delegation

If none apply, suggest an ad-hoc Agent call instead and confirm with the user.
</step>

<step name="design">
Design the subagent definition:

**Frontmatter**:
- name: lowercase-with-hyphens
- description: When Claude should delegate (critical for accuracy)
- tools: minimal necessary set
- model: cheapest that produces reliable results
- maxTurns: if the task has a natural bound
- memory: if cross-session learning is needed
- skills: if project conventions should be preloaded
- hooks: if runtime validation is needed

**System prompt (body)**:
Structure:
1. Role and single objective (first sentence)
2. What the agent receives (inputs)
3. What the agent produces (outputs)
4. Constraints and non-goals
5. Process steps (for complex tasks)
6. Output format specification

Keep the system prompt focused. One agent, one job. If the agent needs to do multiple things, consider splitting into multiple agents or building a skill instead.
</step>

<step name="write">
Write the .md file to the user's specified agents/ directory.

Writing principles:
- Description is the most important field — it determines when delegation happens
- Tool restrictions should be tight enough to prevent drift but loose enough to be useful
- System prompt should state the objective in the first sentence
- Specify output format explicitly so the main agent can consume results
- If using memory, include guidance on what to remember vs what's transient
- Include explicit instruction against `run_in_background: true` — returned output is lost; if the caller needs the result, the call must run in foreground (file output still works)
</step>

<step name="critical-review">
Launch a subagent to review the produced file.

The reviewer checks:
- **System prompt focus**: Does it stay focused on the stated purpose?
- **Tool restrictions**: Not too broad (drift risk), not too narrow (can't do the job)?
- **Description accuracy**: Does it accurately represent what the agent does? Will Claude delegate to it at the right times?
- **Wandering risk**: Any instructions that could cause the agent to pursue adjacent concerns?
- **Output contract**: Is the output format clear enough for the main agent to parse?
- **Model fit**: Is the model choice appropriate for the task complexity?

Report specific issues.
</step>

<step name="review">
Walk through with the user:

1. Present the full subagent definition
2. Explain design choices (model, tools, memory, hooks)
3. Present critical review findings
4. Demonstrate how the main agent would delegate to this subagent

Iterate on feedback.
</step>

<step name="validate">
Checklist:

- [ ] Description is specific enough for accurate delegation
- [ ] Tool restrictions are appropriate
- [ ] System prompt states objective in first sentence
- [ ] Output format is specified
- [ ] maxTurns set if task has natural bound
- [ ] No unnecessary capabilities (principle of least privilege)
- [ ] Memory scope appropriate (or omitted if not needed)
- [ ] Hooks appropriate for risk level
</step>

</process>

<success_criteria>
- Subagent .md file written to specified location
- Justified why custom over ad-hoc
- Description enables accurate delegation
- Tool restrictions match task needs
- System prompt is focused, with clear inputs/outputs
- Critical review passed or issues addressed
- User approved
</success_criteria>
