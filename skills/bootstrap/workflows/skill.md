<required_reading>
Read before starting:
- [reference/skill-format.md](../reference/skill-format.md) — Complete SKILL.md spec and templates
- [reference/enforcement.md](../reference/enforcement.md) — Enforcement mechanism selection
- [reference/prompting.md](../reference/prompting.md) — Prompt engineering techniques

Read if the skill needs subagents:
- [reference/orchestration-patterns.md](../reference/orchestration-patterns.md) — Topology patterns
- [reference/decision-frameworks.md](../reference/decision-frameworks.md) — When to use what
</required_reading>

<process>

## Build a Skill

<step name="interview">
If the interview hasn't been completed in SKILL.md intake, gather requirements now.

Extract via AskUserQuestion (2-4 questions per round, 1-3 rounds):

**Core**:
- What does the skill do? (single sentence)
- What triggers it? (words/phrases the user would say)
- Does it accept arguments? What kind?

**Structure**:
- Single workflow or multiple intents? (simple vs router pattern)
- Does it need subagents for any steps?
- Should it run in forked context or main conversation?

**Enforcement**:
- What's the risk level? (creative / standard / fragile)
- Read-only or can it modify files?
- User-triggered only, or can Claude auto-invoke?

**Context**:
- Personal (~/.claude/skills/) or project (.claude/skills/)?
- Between rounds, explore the codebase if relevant. Populate options from findings, not generic defaults.

Skip questions whose answers are obvious from the user's request.
</step>

<step name="assess">
Based on interview results, classify enforcement level:

**High freedom** (creative tasks: review, analysis, generation):
→ Prompt-level: objective, success criteria, heuristics
→ No tool restrictions (or broad access)
→ No hooks

**Medium freedom** (standard operations: processing, formatting, API calls):
→ Prompt-level: objective, success criteria, preferred patterns
→ allowed-tools: restrict to necessary tools
→ No hooks

**Low freedom** (fragile: migrations, deployments, external side effects):
→ Prompt-level: exact instructions, non-goals, scope boundaries
→ allowed-tools: minimal necessary set
→ hooks: PreToolUse validation, PostToolUse verification
→ Consider disable-model-invocation: true

Present the recommendation to the user. Adjust if they disagree.
</step>

<step name="design">
Choose the structure:

**Simple skill** — single purpose, one workflow:
- Use the simple or multi-step template from skill-format.md
- SKILL.md only, no supporting files

**Argument-based skill** — accepts parameters:
- Use argument-based template
- Set argument-hint in frontmatter

**Forked context skill** — benefits from isolation:
- Set context: fork and agent type
- Use when output is verbose or task is self-contained

**Router skill** — 3+ distinct intents:
- Create SKILL.md with essential_principles, intake, routing
- Create workflows/ directory with one file per intent
- Create reference/ directory if shared knowledge is needed
- Each workflow file has required_reading, process, success_criteria

**Skill with subagents** — needs orchestration:
- Design which subagents are needed
- For each: role, model, tools, input/output contract
- Determine if custom subagents (.claude/agents/) or ad-hoc Agent calls
- Read orchestration-patterns.md for topology selection

If the skill needs custom subagents, offer to also create the subagent definitions.
</step>

<step name="research">
If the skill involves unfamiliar APIs, libraries, or domain conventions:

1. WebSearch for current documentation
2. WebFetch specific pages for technical details
3. Verify findings against the actual codebase if a project is in scope

Embed verified technical details in the skill. Don't rely on Claude's training data for API specifics — it goes stale.
</step>

<step name="write">
Create the skill files:

1. Write SKILL.md with:
   - Frontmatter (name, description, allowed-tools, etc.)
   - XML body (objective, quick_start or workflow, success_criteria)
   - Keep under 500 lines

2. If router pattern: write each workflow file in workflows/
3. If reference material needed: write reference files in reference/
4. If hooks needed: write validation scripts

Write to the user's specified location (personal or project).

### Writing principles
- XML tags for body structure, not markdown headings
- Third-person description with trigger terms
- Single-line YAML strings (no multiline indicators)
- Instructions over explanations — every line should change behaviour
- Match specificity to the assessed freedom level
- Include one concrete example if it clarifies behaviour
- Hardcode structure the skill owns; discover structure the skill consumes (see essential principle 11)
- Explicitly instruct against `run_in_background: true` on Agent and Bash calls — returned output is lost, so the caller cannot act on results (file output still works)
</step>

<step name="critical-review">
Launch a subagent (Agent, subagent_type=general-purpose) to review the produced files.

The reviewer receives:
- The original requirements (from interview)
- All produced files (read each one)

The reviewer checks:
- **Objective alignment**: Does every instruction serve the stated objective?
- **Depth**: Does the skill produce output that requires genuine reasoning? If the entire skill could be replaced by a shell script or existing tooling, the design is too shallow.
- **Scope creep**: Are there instructions addressing concerns the user didn't ask for?
- **Enforcement fit**: Are restrictions matched to the assessed risk level? Not over- or under-specified?
- **Description accuracy**: Will the description trigger reliably for the intended use cases?
- **Progressive disclosure**: Is SKILL.md under 500 lines? Are references one level deep?
- **Anti-patterns**: Any of the listed anti-patterns present?
- **Generality**: Does the skill hardcode project structure it doesn't own? Would it break if invoked from a different project or after a reorganisation?

Report specific issues with file:line references.
</step>

<step name="review">
Walk through the produced files with the user:

1. Present SKILL.md content
2. If router: present each workflow
3. If reference files: summarise their purpose
4. Present any issues from the critical review
5. Explain key design choices (enforcement level, structure, tool restrictions)

Iterate on feedback. Edit files based on user input.
</step>

<step name="validate">
Run the validation checklist:

- [ ] **Trigger test**: Does the description contain the right trigger terms?
- [ ] **Tool restrictions**: Are allowed-tools appropriate for the assessed risk level?
- [ ] **Edge cases**: What happens with empty arguments? Ambiguous input? Missing files?
- [ ] **Anti-drift**: Are there clear scope boundaries and non-goals?
- [ ] **Model compatibility**: Will this work on Haiku/Sonnet/Opus? (Haiku needs more detail)
- [ ] **Progressive disclosure**: SKILL.md under 500 lines? References one level deep?
- [ ] **Hooks**: If low-freedom, are validation hooks in place?
- [ ] **Human checkpoints**: For multi-step skills, are there AskUserQuestion gates?

Present the checklist results to the user. Flag any failures.
</step>

</process>

<success_criteria>
- Skill files written to the specified location
- SKILL.md under 500 lines with correct frontmatter
- Description is third-person with specific trigger terms
- Enforcement level matches assessed risk
- Critical review passed (or issues addressed)
- Validation checklist passed
- User approved the final output
</success_criteria>
