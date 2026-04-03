<workflow>

<step name="load">
**Goal:** Understand the existing design before reviewing it.

Identify the design to review from the user's input:

- If referencing a file or document: read it
- If referencing a PR, issue, or conversation: load it
- If verbal description: ask the user to state the key design decisions

**If the artefact cannot be loaded** (file not found, auth required, inaccessible): stop and tell the user what was attempted and what failed. Do not proceed with incomplete information.

If the design references or is accompanied by an implementation plan, load that too — the plan step may revise it.

Also gather surrounding context:

- What problem does the design solve?
- What constraints was it designed under?

**Check implementation state:** Use Grep and Glob to search for code matching the key design components. Determine whether the design is pre-implementation, partially built, or fully implemented. Report the implementation state — this gates the review depth to what's actually actionable.

If the design references existing systems, delegate broader codebase investigation to a subagent. Use Agent with subagent_type=Explore, model=sonnet, passing the key design components and what context is needed. The subagent returns its findings as its final message, structured under:
- **Related systems**: How referenced systems work, their interfaces and constraints
- **Integration points**: Where the design touches existing code
- **Conventions**: Relevant patterns and standards in the codebase

The subagent requires these tools: Read, Grep, Glob.

**If the subagent returns empty or error results, surface the failure to the user and halt — do not proceed with missing context.**
</step>

<step name="understand">
**Goal:** Confirm understanding before reviewing.

Summarise the design in your own words:

- **Problem**: What it solves
- **Approach**: How it works (key components, data flow, architecture)
- **Key decisions**: Design choices and their rationale
- **Inferences**: Anything that was ambiguous or internally inconsistent in the design and required interpretation to summarise — flag these explicitly rather than silently resolving them

Use AskUserQuestion to confirm:
- Is this understanding correct? (Especially the inferences — are they intended or unresolved?)
- Any specific concerns or areas to focus the review on?

**Checkpoint:** Proceed only when the user confirms your understanding.
</step>

<step name="review">
**Goal:** Critically review the design, surfacing real problems.

Assess against:

1. **Requirements coverage** — Does it solve the stated problem? Gaps?
2. **Failure modes** — What breaks? Edge cases, error paths, scaling limits.
3. **Assumptions** — What's assumed that might be wrong? Dependencies, data shapes, user behaviour.
4. **Simplicity** — Over-engineered? What could be cut without losing capability?
5. **Integration** — Fits with existing codebase? Conflicts with current patterns? Migration concerns.
6. **Completeness** — Unaddressed concerns or undefined behaviours?
7. **Consistency** — Do the parts of the design agree with each other?

Present findings as a numbered list ranked by severity. For each:
- The concern
- Why it matters (concrete impact)
- Suggested mitigation or alternative

Use AskUserQuestion to walk through significant concerns:
- **Accept risk** — Acknowledge and proceed
- **Mitigate** — Adjust the design to address it
- **Rethink** — Explore alternatives for this part of the design

Minor concerns can be noted without requiring a decision.

**If major concerns require a fundamentally different approach:** Offer to switch to the greenfield workflow, carrying forward what's been learned.
</step>

<step name="plan">
**Goal:** Produce an implementation plan incorporating review findings.

If the original design already has a plan, revise it. Otherwise, create one.

Before generating the plan, consolidate the key decisions from the review — design summary, confirmed concerns, accepted risks, and agreed mitigations — into a compact working summary. Reference this consolidated understanding rather than scanning back through the full conversation.

If revising an existing plan, prepend **Changes from original** — what was modified and why.

Follow the plan structure defined in SKILL.md's `<plan_template>`. The report should include: design summary, review findings, decisions, and implementation plan.
</step>

</workflow>

<edge_cases>
- **Design is only a rough idea**: Redirect to the greenfield workflow after confirming with the user.
- **Design is already implemented**: Focus review on discrepancies between design and code. The plan step should address gaps and drift, not re-implement what exists.
- **No concerns found**: State that the design looks sound. Still produce an implementation plan if one doesn't exist.
- **User disagrees with a concern**: Drop it and move on. Note the disagreement but don't argue.
- **User stops early**: Summarise findings so far, note unresolved concerns.
</edge_cases>
