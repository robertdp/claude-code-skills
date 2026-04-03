<required_reading>
Read before starting:
- [reference/orchestration-patterns.md](../reference/orchestration-patterns.md) — Topology patterns
- [reference/decision-frameworks.md](../reference/decision-frameworks.md) — When to use what
- [reference/enforcement.md](../reference/enforcement.md) — Enforcement for each agent
</required_reading>

<process>

## Design an Orchestration Topology

<step name="interview">
Extract via AskUserQuestion (2-4 questions per round):

**Goal**:
- What should the system accomplish end-to-end?
- What are the distinct phases or dimensions?
- Which phases depend on which?

**Parallelism**:
- Can any phases run in parallel?
- Do parallel workers need the same input (fan-out) or different partitions?
- Is there a synthesis step after parallel work?

**State**:
- How does state flow between agents? (Documents on disk, inline summaries, structured data)
- Is there shared context all agents need? (Codebase map, project conventions)
- Where should context be cleared vs carried forward?

**Constraints**:
- Budget constraints? (Model selection, number of agents)
- Latency constraints? (Foreground vs background)
- Quality requirements? (Validation between phases)
</step>

<step name="feasibility">
Before selecting a multi-agent pattern, assess:

1. **Decomposability**: Can the task be split into subtasks that execute independently? If subtasks have high sequential dependency (each step requires the full output of the previous), coordination will degrade performance — use a single agent with a pipeline of steps instead of a pipeline of agents.
2. **Single-agent adequacy**: Could one agent handle this with sequential tool calls? If yes, the coordination overhead isn't justified.
3. **Tool density**: How many distinct tools does each subtask use? Tool-heavy subtasks suffer more from multi-agent overhead.

If the task fails these checks, recommend a single-agent approach and confirm with the user before proceeding.
</step>

<step name="pattern-match">
Identify which orchestration pattern(s) fit. Present options with tradeoffs.

**Hub-and-spoke**: Coordinator delegates to specialists, synthesises.
→ When: Multiple independent analyses of the same input.

**Fan-out / fan-in**: Same operation on different partitions.
→ When: Batch processing, multi-repo analysis, parallel test suites.

**Pipeline**: Sequential phases, documents as interfaces.
→ When: Each phase needs the previous phase's output.

**Router**: Single entry dispatching to different workflows.
→ When: One skill handles multiple distinct intents.

**Hybrid**: Combinations (e.g., pipeline with fan-out at one stage).
→ When: Different stages have different structures.

Use AskUserQuestion to confirm the pattern selection. If the user is unsure, recommend based on the task structure.
</step>

<step name="design-topology">
For each agent in the topology, define:

1. **Role**: What it does (single sentence)
2. **Model**: Haiku (fast/cheap), Sonnet (balanced), Opus (complex reasoning)
3. **Tools**: Minimal necessary set
4. **Input contract**: What it receives (format, source)
5. **Output contract**: What it returns (format, size — aim for 1-2k token summaries)
6. **Enforcement level**: Based on risk assessment
7. **Depth check**: Does this agent's work require genuine judgment? If it could be fully replicated by grep, regex, or existing tooling, widen the scope to include analysis that requires reasoning about context, relationships, and trade-offs.

### Agent type decision

For each agent, evaluate against the custom subagent criteria from `decision-frameworks.md`:

| Criterion | Applies? |
|-----------|----------|
| Reused across multiple sessions or skills | |
| Needs specific tool restrictions enforced consistently | |
| Benefits from persistent memory | |
| Needs skills preloaded | |
| Needs lifecycle hooks | |
| Benefits from context isolation with a well-defined system prompt | |
| Description must be precise for accurate delegation | |
| Needs explicit permissions (background write work) | |

**If any criterion applies**: define as a custom subagent (`.claude/agents/`). Write the agent definition as part of the artefacts in the write step.

**If none apply**: use an ad-hoc Agent call with an inline prompt.

**Do not default to ad-hoc Agent calls for convenience.** The evaluation must be explicit for each agent in the topology. State which criteria apply and which don't.

Draw an ASCII topology diagram showing agent relationships and data flow.
</step>

<step name="context-strategy">
Apply the context strategy from `orchestration-patterns.md` to each agent boundary. For each transition decide: clean-slate isolation, shared context, context clearing, or shared artefact injection.
</step>

<step name="write">
Generate all artefacts:

1. **Topology diagram** (ASCII) with agents, flows, and data labels
2. **Custom subagent definitions** (`.claude/agents/*.md`) for every agent that passed the type decision gate — do not defer these to "later"
3. **Ad-hoc Agent prompts** (inline in the skill) only for agents where no custom subagent criteria applied
4. **Skill files** (if the orchestration is wrapped in a skill)
5. **Communication contracts** (what each agent receives and returns)
6. **State management** (where documents/data live between phases)

Write to the user's specified location. Custom subagents go in the same scope as the skill (personal or project).
</step>

<step name="critical-review">
Launch a subagent to review the topology.

The reviewer checks:
- **Problem-solution fit**: Does the topology actually solve the stated problem?
- **Depth**: Does each agent do substantive work requiring judgment? Would a domain expert find the output valuable, or is it surface-level observation that existing tools already produce?
- **Agent type decisions**: Are custom subagents used where the criteria apply? Are ad-hoc Tasks justified where used? Evaluate each agent against the criteria from `decision-frameworks.md`.
- **Agent necessity**: Are all agents needed? Can any be merged?
- **Context strategy**: Is isolation used where it helps? Shared context where needed?
- **Communication contracts**: Are they clear and consistent?
- **Single points of failure**: What happens if one agent fails?
- **Cost efficiency**: Are cheap models used where possible?
</step>

<step name="review">
Walk through the topology with the user:

1. Present the ASCII diagram
2. Walk through each agent's role and configuration
3. Explain the context strategy
4. Present critical review findings
5. Stress-test: "What happens if agent X returns garbage?"

Iterate on feedback.
</step>

</process>

<success_criteria>
- Topology diagram produced
- All agent definitions written
- Communication contracts defined
- Context strategy documented
- No unnecessary agents
- Critical review passed or issues addressed
- User approved
</success_criteria>
