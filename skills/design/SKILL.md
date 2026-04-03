---
name: design
description: Research, design, and plan before building. Structured problem exploration, option evaluation, critical review, and implementation planning. Use when user says "research", "design", "plan the approach", "think through", "explore options", or wants to go from problem to plan. Invoked via /design or /robert:design.
argument-hint: "<problem, goal, or existing design to review>"
allowed-tools: Read, Grep, Glob, Agent, WebSearch, WebFetch
---

<objective>
Go from problem to plan. Research a problem space, explore solution options, validate through critical review, and produce an actionable implementation plan. Two modes: greenfield (start from a problem statement) and review (stress-test an existing design or plan).
</objective>

<essential_principles>
- **Understand before proposing.** Read code, research context, verify claims. Never design around assumptions you could resolve with investigation.
- **Apply domain expertise.** Bring relevant knowledge of patterns, pitfalls, and prior art to the problem. If the user is heading toward a known antipattern or a solution that doesn't fit the scale/context, say so directly. Guide toward the optimal solution; steer away from sub-optimal ones.
- **Informed recommendation, not neutral menu.** Where evidence points to a clear best option, say so and explain why. Present alternatives when the tradeoff is genuinely context-dependent, but do not present weak options as equally viable. The user makes the final call, but you bring the expertise.
- **Validate before committing.** Look for failure modes, wrong assumptions, and missing requirements before finalising any design.
- **Concrete over abstract.** Specifics, file references, examples, numbers. "This won't scale" is useless. "This does O(n²) lookups on the users table, which has 2M rows" is actionable.
- **User controls the process.** Checkpoint at each phase transition. The user can skip ahead, redirect, or stop at any point.
</essential_principles>

<quick_start>
1. Determine mode: greenfield (new problem) or review (existing design)
2. Follow the appropriate workflow
3. End with an actionable implementation plan
</quick_start>

<intake>
**User input:**

<input>
$ARGUMENTS
</input>

Determine the mode from the input:

- **Greenfield**: The user wants to explore a problem, build something new, or redesign/replace an existing system.
- **Review**: The user explicitly wants to evaluate or stress-test an existing design, plan, or architecture — not just mentions one.

**Disambiguation examples:**
- "Add authentication to our API" → greenfield (new capability)
- "Redesign the auth system" → greenfield (replacement, not review of existing)
- "Review our caching design doc" → review (explicit evaluation intent)
- "I'm thinking of using event sourcing for audit logs" → greenfield (exploring a new approach)

After determining mode, state it in one line before loading the workflow (e.g., "Starting in greenfield mode — exploring a new solution. Correct?"). If the user corrects, switch workflows immediately.

</intake>

<routing>
| Mode | Workflow |
|------|----------|
| Greenfield (new problem/feature) | [workflows/greenfield.md](workflows/greenfield.md) |
| Review (existing design/plan) | [workflows/review.md](workflows/review.md) |

Read the selected workflow file and follow it exactly.
</routing>

<plan_template>
The plan structure below is shared by both workflows. Workflow-specific additions (consolidation context, "Changes from original") are specified in each workflow's plan step.

**Overview** — 2-3 sentence summary of the approach and key decisions.

**Steps** — Ordered list of implementation steps. For each:
- What to do (specific files, functions, components)
- Why (which requirement or design decision this serves)
- Dependencies (what must be done first)
- Done when (verifiable acceptance criterion — e.g., tests pass, API contract satisfied, observable behaviour X)

**Risks & mitigations** — Concerns from validation or review, with mitigations where applicable.

**Open questions** — Anything that couldn't be resolved and needs to be answered during implementation.

Keep the plan concrete — reference specific files, functions, and patterns from the codebase. No generic steps like "implement the feature".

**Checkpoint:** Present the plan. The user can approve, request changes, or go back to any previous step.

After approval, use AskUserQuestion to offer: write the full report to a file for future reference. If accepted, write to a user-specified path or a sensible default.
</plan_template>

<success_criteria>
**Both workflows:**
- Risks and open questions surfaced explicitly
- Actionable implementation plan produced
- User retained control at every checkpoint

**Greenfield:**
- Problem clearly understood before solutions explored
- Multiple options evaluated with concrete tradeoffs
- Chosen design validated through critical review

**Review:**
- Existing design accurately understood and confirmed
- Concerns surfaced, triaged, and resolved (accept / mitigate / rethink)
- Plan incorporates review findings and agreed mitigations
</success_criteria>
