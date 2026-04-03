---
name: phase
description: Break a spec or requirements into a phased implementation plan with self-contained files for minimal-context agent execution. Use when user says "decompose", "break down", "phase plan", "staged implementation", or invokes /phase or /robert:phase.
argument-hint: "[spec-file-path]"
allowed-tools: Read, Write, Edit, Grep, Glob, Bash(git *), Bash(gh *), Agent
model: opus
---

Current branch: !`git branch --show-current`

<objective>
Take a feature spec (or requirements gathered through interview) and generate a phased implementation plan as a set of self-contained files. The generated files enable **minimal-context execution**: any agent can read the master file and a phase file, then implement that phase without prior conversation history.

Generated artifacts:
- **Master file** (PLAN.md) — Goals, constraints, phase sequence, agent instructions
- **Phase files** (phase-NN-name.md) — Self-contained scope, acceptance criteria, verification
- **State file** (state.json) — Progress tracking for coordination between sessions
</objective>

<quick_start>
`/phase specs/my-feature.md` — Generate a phased plan from a spec file.
`/phase` — No spec file? Interview to capture requirements first.

Generated files are committed as the first commit on a working branch created via the `worktree` skill (`/worktree` or `/robert:worktree`).
</quick_start>

<workflow>
<step name="input">
<user-input>
$ARGUMENTS
</user-input>

Parse the user input for a file path.

- **File path given**: Read the spec file. Verify it contains enough detail to decompose (goals, requirements, constraints). If too vague, ask user to clarify before proceeding.
- **No file path**: Interview the user to capture requirements. Ask about: goals, scope, constraints, known files/modules affected, acceptance criteria. Gather enough detail to generate a meaningful plan.
</step>

<step name="explore">
Investigate the codebase to ground the plan in reality. For large or unfamiliar codebases, use Agent with subagent_type=Explore to delegate deep investigation. For smaller or familiar codebases, use tools directly:

1. **Project structure**: Use Glob to find `package.json`, `tsconfig.json`, build configs, and directory layout. Read key config files to understand the build system, dependencies, and module boundaries.
2. **Affected files**: Use Grep to search for types, functions, and modules mentioned in the spec. Read the files that will need changes. List them explicitly — these become the "Files to Create/Modify" in phase files.
3. **Patterns and conventions**: Find an existing file similar to what the plan will create. To find it: Grep for a key type or function name from the spec, then read a file in the same directory or module. For example, if the spec adds a new API route, Grep for an existing route handler (`router.get`, `app.post`, etc.) and read that file. Note naming conventions, file structure, test patterns.
4. **Test infrastructure**: Use Glob to find test directories and config (`jest.config.*`, `vitest.config.*`, `*.test.*`). Note the test runner and assertion style.

Output of this step: a concrete list of files to touch, grouped by area of concern, with notes on dependencies between them.
</step>

<step name="decompose">
Break the work into phases. Read [reference/templates.md](reference/templates.md) for the file formats you will generate.

**Sizing principle:** Agent output quality degrades as context fills up. Keep phases small enough that an agent can read the relevant code, hold the full implementation in mind, and complete the work without running out of context. This also keeps commits focused and reviewable.

- Aim for **~8-10 files per phase** as a guideline. Adjust based on complexity — many simple files is fine, fewer complex files may still need splitting.
- Avoid large, complex, multi-file work in a single phase when it can be split.

**Ordering:** Phases are strictly sequential — each phase depends on all previous phases being complete. Order them so later phases build on earlier ones. When a phase builds on files created or modified by an earlier phase, reference those files explicitly in the later phase's file list or Context & Notes. The executing agent should not have to read earlier phase files to understand its own scope.

**Finalize phase:**
Add a final finalize phase when either condition is met:
- Any file appears in the "Files to Create/Modify" list of 2+ phases (cross-phase overlap)
- The plan has 5 or more phases (backstop)

This phase:
- Diffs the final implementation against the original spec — flags missed requirements and unintended additions
- Identifies bugs introduced across phase boundaries
- Checks for consistency (naming, patterns, error handling) across all phases
- Simplifies where phases introduced redundant or over-complex code
- Runs the full test/build/lint suite
- After verification passes: deletes the plan files directory as a separate final commit
</step>

<step name="validate-coverage">
Produce a **coverage table** mapping every spec requirement to the phase that addresses it. Present this table to the user — do not silently self-validate.

| Requirement | Phase | Notes |
|-------------|-------|-------|
| {requirement from spec} | {phase #} | |
| {requirement from spec} | — | **GAP**: not covered |

1. List every requirement, constraint, and acceptance criterion from the spec
2. For each one, identify which phase addresses it
3. Flag any requirement not covered by at least one phase (mark as GAP)
4. Flag any phase that doesn't trace back to a spec requirement (mark as SCOPE CREEP)

Show this table to the user immediately. If gaps exist, revise the phase breakdown before proceeding. Do not generate files for an incomplete plan.
</step>

<step name="critique">
Use Agent with subagent_type=general-purpose to adversarially review the plan. Pass the subagent:
- The original spec (or interview notes)
- The proposed phase breakdown (phase names, scopes, file lists, ordering)
- The coverage table from the previous step

Frame the subagent prompt explicitly: "Assume this plan has problems. Find them. Only report issues that would cause a phase to fail, produce wrong results, or block a later phase. Do not report stylistic preferences or theoretical risks. For each issue, state it as a fact — not a suggestion."

The subagent must answer these pass/fail checks:
- **Dependencies**: Does phase N produce everything phase N+1 needs? For each phase, verify that its inputs exist in earlier phases' outputs or the existing codebase.
- **Boundaries**: Are cross-cutting concerns (auth, error handling, logging, types) addressed where they're used, or silently deferred to a phase that doesn't know about them?
- **Assumptions**: Does any phase assume something that isn't in an earlier phase's scope, acceptance criteria, or the existing codebase?
- **Sizing**: Can an agent realistically implement each phase in a single session without running out of context?
- **Feasibility**: Given what the explore step found about the codebase, is each phase's file list and scope realistic?

Each issue gets a severity:
- **blocking** — phase will fail or produce wrong results. Must be resolved before generating files.
- **warning** — will degrade quality or cause rework. Record in the relevant phase file's Context & Notes section.

Present the subagent's findings to the user. Revise the plan if any blocking issues exist.
</step>

<step name="generate">
Pick a default location for plan files: `plans/[feature-name]/` at the repo root (or match existing `plans/` directory if one exists). Generate all plan files there using the templates in [reference/templates.md](reference/templates.md). The user can change the location during the review step.

1. **Master file** (PLAN.md):
   - Condensed summary of goals and constraints (from spec or interview)
   - Phase index table
   - Agent instructions section (how to pick up and work on a phase)

2. **Phase files** (phase-01-name.md, phase-02-name.md, ...):
   - Reference to master file for overall context
   - Specific scope for this phase
   - Files to create/modify with brief descriptions of what to do
   - Acceptance criteria (specific, verifiable)
   - Optional verification step (tests, build, linter)
   - Context and notes the agent needs

3. **State file** (state.json):
   - Per-phase status tracking (pending/in_progress/completed)
   - Space for agent observations

Every phase file must be **self-contained**: an agent with minimal context (master file + phase file) should have everything needed to implement the work.
</step>

<step name="review">
**Present the plan to the user before committing.** Show:

- The phase index table (from the master file)
- Total phase count

Use AskUserQuestion to confirm the user is satisfied, or gather feedback to revise. The user may also change the file location at this point. Do not proceed to the commit step until the user approves.

**PR strategy:** Ask the user when they want PRs opened. For small plans (2-3 phases), a single PR after the finalize phase is typical. For larger plans with milestones, PRs at milestone boundaries make sense. Record the PR strategy in the master file so executing agents know when to open PRs.
</step>

<step name="commit">
Use the `worktree` skill (`/worktree` or `/robert:worktree`) to create a working branch, then commit the plan files:

1. Create a working branch using the worktree skill
2. Add all generated plan files
3. Commit with message: `Plan: [feature name] - N phases`
4. Report the branch name, file locations, and phase summary to the user

**Commit sequence for the full plan lifecycle:**
- **Commit 0** (this step): Plan files only (PLAN.md, phase files, state.json)
- **Commit 1**: Phase 1 implementation + state.json updated to completed
- **Commit 2**: Phase 2 implementation + state.json updated
- ...
- **Commit N-1**: Finalize phase - verification fixes, consistency cleanup
- **Commit N**: Delete plan files directory (only after verification passes)
</step>

<step name="handoff">
Output a copy-pasteable bootstrap prompt the user can paste into a new session to start phase 1:

```
Implement the next phase of plans/{feature-name}/PLAN.md
```

This same prompt is used for every phase — the agent reads state.json to determine which phase is next. The prompt is output:
- Here, after the plan is generated and committed
- After each phase completes (the executing agent outputs it as its final action)
</step>
</workflow>

<edge_cases>
- **Spec too vague**: Ask the user for more detail before decomposing. A plan built on vague requirements produces bad phases.
- **Tiny spec (1-2 phases)**: Still generate the full file structure. Consistency matters for the workflow. A 2-phase plan with proper files is better than ad-hoc implementation.
- **Existing plan files at location**: Warn the user and ask whether to overwrite or choose a new location.
- **No git repo**: Skip the commit step. Generate files in the requested directory and warn that commit-based progress tracking won't work.
- **Very large spec (20+ phases)**: Group phases into milestones (see milestone section in [reference/templates.md](reference/templates.md)). Each milestone is a cluster of related phases. Milestone boundaries are where branches/PRs may split.
- **Spec changes after plan exists**: If a plan already exists and the spec has changed, read the existing plan, diff against the new spec, and propose targeted updates to affected phases rather than regenerating from scratch. Ask the user whether to update in-place or regenerate.
- **Phase fails**: Stop and report what went wrong and what was attempted. Output the bootstrap prompt so the user can decide next steps. Do not attempt rollback or state updates.
</edge_cases>

<success_criteria>
- User approved the plan before it was committed
- Working branch exists (created via worktree skill) with plan files as the first commit
- Master file accurately summarizes the spec's goals and constraints
- Each phase file is self-contained — an agent with minimal context can read master + phase file and implement the work
- Phase sizing follows the principle: small enough to implement in one session, large enough to be meaningful
- state.json has all phases in "pending" status
- User has the bootstrap prompt to start phase 1
</success_criteria>
