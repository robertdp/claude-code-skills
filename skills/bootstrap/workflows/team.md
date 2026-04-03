<required_reading>
Read before starting:
- [reference/teams-reference.md](../reference/teams-reference.md) — Agent teams spec
- [reference/decision-frameworks.md](../reference/decision-frameworks.md) — Teams vs subagents
- [reference/orchestration-patterns.md](../reference/orchestration-patterns.md) — Topology patterns
</required_reading>

<process>

## Build an Agent Team Configuration

<step name="interview">
Extract via AskUserQuestion (2-4 questions per round):

**Purpose**:
- What should the team accomplish?
- Why do teammates need to communicate? (If they don't, suggest subagents instead.)
- What are the distinct specialisations?

**Structure**:
- How many teammates? (Start with 3-5)
- What does each specialise in?
- Which files does each own? (Avoid overlap)
- How many tasks per teammate? (Target 5-6)

**Quality**:
- What quality gates are needed?
- Should the lead approve plans before implementation?
- What happens if a teammate produces bad output?

**Infrastructure**:
- Display mode preference? (in-process, tmux, auto)
- Permission mode for teammates?
</step>

<step name="justify">
Justify why agent teams over orchestrated subagents:

**Teams are right when** (check at least one):
- [ ] Workers need to share findings and challenge each other
- [ ] Cross-layer coordination (frontend + backend + tests)
- [ ] Debugging with competing hypotheses
- [ ] Research requiring genuinely different perspectives
- [ ] Inter-agent communication adds value exceeding the cost

**Cost reality**: Coordination overhead scales super-linearly with team size — each additional teammate adds more overhead than the last, not a flat multiplier. A 5-person team is not 5× the cost of one agent; it's worse. The task must also be genuinely decomposable — high sequential dependency between teammates degrades results regardless of team size.

If none of the above apply, or the task isn't decomposable, suggest subagents or a single agent. Present cost comparison.
</step>

<step name="design">
Design the team:

1. **Team lead role**: What the lead coordinates. The lead should manage, not implement.
2. **Teammate definitions**: For each teammate:
   - Name and specialisation
   - File ownership boundaries (must not overlap)
   - Spawn prompt (task-specific context)
   - Permission mode
3. **Task structure**: Break work into 5-6 tasks per teammate with clear dependencies.
4. **Quality gates**:
   - TeammateIdle hook: What check runs when a teammate is about to go idle?
   - TaskCompleted hook: What validation prevents premature completion?
5. **Communication strategy**: When to message vs broadcast. Broadcast sparingly.
</step>

<step name="write">
Generate:

1. Team configuration guidance (how to spawn the team)
2. Teammate definitions (if custom agents needed)
3. Hook scripts (TeammateIdle, TaskCompleted) if quality gates needed
4. Task breakdown template

Remind the user they must set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in their settings.json env block.
</step>

<step name="critical-review">
Launch a subagent to review.

The reviewer checks:
- **Specialisation alignment**: Does each teammate's focus match the stated purpose?
- **File conflicts**: Are ownership boundaries clear enough?
- **Quality gates**: Are hooks appropriate for the risk level?
- **Cost justification**: Does the coordination value exceed the ~2-5x cost?
- **Task granularity**: Are tasks the right size? (Not too big, not too small)
</step>

<step name="review">
Walk through with the user:

1. Present team structure and specialisations
2. Present task breakdown
3. Explain quality gates
4. Present critical review findings
5. Estimate cost multiplier

Iterate on feedback.
</step>

<step name="validate">
Checklist:

- [ ] Justified why teams over subagents
- [ ] File ownership boundaries are clear
- [ ] No file overlap between teammates
- [ ] Quality gate hooks defined
- [ ] Task dependencies are acyclic
- [ ] 5-6 tasks per teammate
- [ ] Lead manages, doesn't implement
- [ ] Cost acknowledged
</step>

</process>

<success_criteria>
- Team configuration documented
- Teammate definitions written if custom agents needed
- Hook scripts written if quality gates needed
- Justified over subagents
- File ownership clear, no conflicts
- Critical review passed or issues addressed
- User approved and understands cost implications
</success_criteria>
