<workflow>

<step name="interview">
**Goal:** Understand the problem before exploring solutions.

Use AskUserQuestion with 2-4 questions per round. Use single-select for decisions where exactly one path must be chosen. Use multiSelect when multiple options can apply simultaneously. Progress through:

1. **Context & Goals** — What problem are we solving? Why now? What does success look like?
2. **Constraints** — Non-negotiables, tech stack limitations, compatibility requirements, timeline.
3. **Scope** — What's in, what's explicitly out? MVP vs full vision.
4. **Assumptions** — Look for gaps, contradictions, and incorrect information in what's been gathered. When you find a conflict between a user answer and codebase evidence, present both sides and ask which is current. Surface unstated dependencies and risks.

Between rounds, gather context autonomously:

- Read relevant codebase files (configs, schemas, entry points, existing implementations)
- Use Agent with subagent_type=Explore for broader codebase investigation
- Use WebSearch when the domain or tooling is unfamiliar

Only ask the user questions you can't answer by reading code. Intent, priorities, and business context come from the user. File structure, dependencies, patterns, and tech stack come from investigation.

**Verify, don't assume**: When a user answer can be checked against the codebase or external sources, check it. Confident answers are not necessarily correct answers.

**Backtracking**: If an answer contradicts or invalidates an assumption from an earlier phase — or conflicts with what you found in the codebase — revisit that phase before continuing. State what changed and re-confirm.

**Stop when:** A round surfaces no new unknowns, constraints, or tradeoffs. Most interviews complete in 2-3 rounds.

**Checkpoint:** Summarise what you've gathered — goals, constraints, scope, key findings from investigation. Confirm with the user before proceeding.
</step>

<step name="research">
**Goal:** Map the solution landscape before committing to an approach.

**Delegate to subagent** to keep the main context clean. Use Agent with subagent_type=Explore, passing:
- The confirmed interview summary (goals, constraints, scope)
- Specific investigation questions based on what the interview surfaced

The subagent should investigate:
- **Codebase patterns**: How does the codebase handle similar problems? What conventions exist?
- **Prior art**: Established patterns, libraries, or approaches in this domain (use WebSearch).
- **Technical feasibility**: What APIs, services, or infrastructure already exist that could be leveraged?
- **Risks**: What could go wrong? What's hard? What's unknown?

The subagent returns its findings as its final message, structured under these categories:
- **Existing patterns**: How the codebase handles similar problems, conventions found
- **Prior art**: Established approaches, libraries, or solutions in this domain
- **Constraints discovered**: Technical limitations, dependencies, infrastructure boundaries
- **Risks**: What could go wrong, what's hard, what's unknown

The subagent requires these tools: Read, Grep, Glob, WebSearch, WebFetch.

**If the subagent returns empty or error results on a non-trivial target, re-run once. If both attempts fail, surface the failure to the user and halt — do not proceed with missing research.**

Present the findings to the user.

**Checkpoint:** Use AskUserQuestion to confirm before proceeding to design. The research may have changed the problem framing — ask whether the scope or direction should change, or whether any findings warrant deeper investigation.
</step>

<step name="design">
**Goal:** Explore options, evaluate tradeoffs, converge on an approach.

Use the scope confirmed at the research checkpoint. If scope changed since the interview, base options on the updated framing, not the original interview summary.

Generate 2-4 concrete solution options. For each:

- **Name**: One-line summary
- **How it works**: Architecture, key components, data flow
- **Tradeoffs**: Pros and cons as a bullet list (prefix + or -)
- **Fits current context**: Does this fit the stated constraints? Why or why not.

If one option is clearly best given the constraints and codebase, lead with your recommendation and explain the reasoning. Include alternatives for completeness, but label them as such (e.g., "consider if X constraint changes"). Do not present weak options as equally viable.

Ground options in what you found during research. Reference existing codebase patterns, libraries, or infrastructure.

Use AskUserQuestion (single-select) to present the options. Place the recommended option first. Include a hybrid option if elements combine well.

If the user wants to explore an option deeper before deciding, do so — read more code, research more, then present updated tradeoffs.

**Checkpoint:** Confirm the chosen approach before proceeding to validation.
</step>

<step name="validate">
**Goal:** Stress-test the chosen design before committing to a plan.

**Delegate to subagent** to get an independent critical review. Use Agent with subagent_type=general-purpose, passing:
- The chosen design (approach, architecture, key components)
- Goals and constraints from the interview
- Research findings from the previous step

The subagent's brief: "Assume this design is wrong. Find the strongest argument against proceeding." It checks:

1. **Requirements fit** — Does it actually solve the stated problem? What's missing?
2. **Risky assumptions** — What are we assuming that might be wrong? What happens if each assumption fails?
3. **Complexity** — Is there a simpler way to achieve the same result? What could be cut?
4. **Integration** — Conflicts with existing codebase patterns? Migration path? Breaking changes?
5. **Failure modes** — What breaks under load, bad input, or partial failure?

The subagent requires these tools: Read, Grep, Glob, WebSearch, WebFetch.

The subagent returns its findings as its final message. For each concern:
- What the concern is
- Concrete impact if left unaddressed
- Suggested mitigation or adjustment

**If the subagent returns empty or error results, surface the failure to the user and halt — do not proceed with missing validation. This is the highest-risk failure point: proceeding without validation produces a false "no concerns" signal.**

Apply severity thresholds:
- **Dealbreaker**: Invalidates the core approach — return to the design step with the new constraint.
- **Significant**: Would cause rework of more than one plan step if left unaddressed — present to the user via AskUserQuestion before proceeding.
- **Minor**: Foldable into the plan as a risk or implementation note.
</step>

<step name="plan">
**Goal:** Produce an actionable implementation plan.

Before generating the plan, consolidate the key decisions from interview, research, design, and validation into a compact working summary: goals, constraints, chosen approach, confirmed mitigations. Reference this consolidated understanding rather than scanning back through the full conversation.

Follow the plan structure defined in SKILL.md's `<plan_template>`. The report should include: interview summary, research findings, design decision, validation concerns, and implementation plan.
</step>

</workflow>

<edge_cases>
- **Problem is too vague**: Ask one broad scoping question before starting the interview phases.
- **Only one viable approach**: Explicitly state the key assumption it depends on and ask the user to confirm it holds. Do not fabricate a weak alternative just for comparison.
- **User wants to skip research**: Proceed to design, but note that options are based on current knowledge only.
- **Validation surfaces a dealbreaker**: Return to design. State what changed and why.
- **User stops early**: Summarise what's been decided so far and list remaining unknowns.
</edge_cases>
