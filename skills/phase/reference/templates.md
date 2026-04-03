# Plan File Templates

Use these templates when generating plan files. Adapt content to the specific feature but preserve the structure — agents depend on consistent formatting to parse and navigate the plan.

## Master File (PLAN.md)

```markdown
# Plan: {Feature Name}

> **Source**: {path/to/spec.md | "Interview"}
> **Created**: {YYYY-MM-DD}

## Goals

{1-3 paragraphs — enough for any agent to understand the vision without reading the original spec.}

## Phases

| # | Phase |
|---|-------|
| 1 | {Phase name} |
| 2 | {Phase name} |
| 3 | {Phase name} |
| 4 | {Phase name} |

Phases are executed sequentially in order. Each phase is a single commit.

**PR strategy:** {e.g., "Single PR after finalize" or "PR after each milestone".}

## Commit Sequence

- **Commit 0**: Plan files (this commit — PLAN.md, phase files, state.json)
- **Commit 1**: Phase 1 implementation + state.json updated to completed
- **Commit 2**: Phase 2 implementation + state.json updated
- ...
- **Commit N-1**: Finalize - verification fixes, consistency cleanup
- **Commit N**: Delete plan files directory

## Working with This Plan

### Starting a phase

1. Read this file (PLAN.md) for overall context
2. Read `state.json` — find the first phase with status `"pending"`
3. Read the corresponding phase file (e.g., `phase-01-setup-schema.md`)
4. Implement the work described in the phase file
5. Verify acceptance criteria are met

### Completing a phase

1. Confirm all acceptance criteria from the phase file are satisfied
2. Run any verification steps listed in the phase file
3. Update `state.json`: set the phase status to `"completed"`, add any notes
4. Commit all changes (implementation + state.json update) as a single commit
5. **Stop.** Output the bootstrap prompt below so the user can start the next phase in a fresh session.

### Bootstrap prompt

Output this after every phase completion:

```
Implement the next phase of {path}/PLAN.md
```

### Handling failures

If a phase cannot be completed:
1. **Stop.** Report what went wrong and what was attempted.
2. Output the bootstrap prompt so the user can decide next steps.
3. Do not commit partial work. Do not attempt rollback.
```

---

## Phase File (phase-NN-name.md)

```markdown
# Phase {N}: {Phase Name}

> **Plan**: Read [PLAN.md](PLAN.md) for overall goals and context.

## Scope

{What this phase accomplishes and why it's separate.}

## Files to Create/Modify

- `path/to/file1.ts` — {what to do: create new file, add function X, modify Y}
- `path/to/file2.ts` — {what to do}
- `path/to/file3.test.ts` — {what to do}

## Acceptance Criteria

- [ ] {Specific, verifiable criterion}
- [ ] {Another criterion}
- [ ] {Tests pass / build succeeds / specific behavior works}

## Verification

```bash
{project's test/build commands — omit section if no meaningful automated check}
```

## Context & Notes

{Gotchas, planning rationale, relevant docs, patterns to follow (with file:line references). Reference specific files from earlier phases if this phase builds on them.}
```

---

## Finalize Phase (phase-NN-finalize.md)

Auto-generated when any file appears in 2+ phases, or when the plan has 5+ phases.

```markdown
# Phase {N}: Finalize

> **Plan**: Read [PLAN.md](PLAN.md) for overall goals and context.

## Scope

Critically verify the implementation against the original spec, then clean up plan artifacts. Approach verification as a pragmatic adversarial review — actively look for gaps, bugs, and drift rather than confirming things work.

## Checklist

### Verify

Assume nothing works until proven otherwise. Read the code, trace the logic, check the claims.

- [ ] **Spec coverage**: Walk each spec requirement and find the code that implements it. Flag requirements with no corresponding code and code with no corresponding requirement.
- [ ] **Bug scan**: Use `git diff main..HEAD -- {file}` for each file touched across phases. Look for logic errors, missing error handling, broken edge cases, and issues at phase boundaries where one phase's assumptions may not match another's output.
- [ ] **Consistency**: Naming, patterns, and error handling are uniform across all phases. Flag divergences introduced by different phases solving similar problems differently.
- [ ] **Simplification**: Identify redundant code, over-abstractions, or workarounds that can be collapsed now that all phases are complete. Cross-phase duplication is common — look for it.
- [ ] **Tests pass**: Full test suite, build, and lint pass.

### Clean up

Only after all verification checks pass:

- [ ] Delete the plan files directory (PLAN.md, phase files, state.json)
- [ ] Commit the deletion as a separate final commit
- [ ] Ensure no WIP commits remain in the branch history

Plan files remain in the git history (commit 0) for future reference.

## Verification

```bash
{project's test command}
{project's build command}
{project's lint command}
```

## Context & Notes

Review the `notes` field in state.json for each completed phase — these capture decisions and deviations made during implementation. Use them to identify where drift is most likely.
```

---

## State File (state.json)

Phases are executed sequentially in order. The first phase with status `"pending"` is the next to work on.

```json
{
  "plan": "{Feature Name}",
  "spec": "{path/to/spec.md or null}",
  "created": "{YYYY-MM-DD}",
  "phases": [
    {
      "id": 1,
      "name": "{Phase Name}",
      "file": "phase-01-{slug}.md",
      "status": "pending",
      "notes": []
    },
    {
      "id": 2,
      "name": "{Phase Name}",
      "file": "phase-02-{slug}.md",
      "status": "pending",
      "notes": []
    },
    {
      "id": 3,
      "name": "{Phase Name}",
      "file": "phase-03-{slug}.md",
      "status": "pending",
      "notes": []
    },
    {
      "id": 4,
      "name": "{Phase Name}",
      "file": "phase-04-{slug}.md",
      "status": "pending",
      "notes": []
    }
  ]
}
```

### Status values

Only `pending` and `completed` are committed. The other states exist at runtime only.

| Status | Meaning | Committed? |
|--------|---------|------------|
| `pending` | Not started. | Yes |
| `in_progress` | An agent is currently working on this phase. | No — runtime only |
| `completed` | Phase is done. Acceptance criteria met. | Yes |
| `failed` | Phase was attempted but could not be completed. | No — runtime only |

### Notes field

Array of strings. Each entry is one observation or event — one sentence max, append only. Examples:

- **completed**: `"Chose JWT over session tokens — see phase file for rationale"`
- **in_progress**: `"Taking the migration approach from phase-01 context notes"`

---

## Milestones (20+ phase plans)

For very large plans, group phases into milestones in the PLAN.md phase index. Milestone boundaries are where branches and PRs may split — each milestone can be its own branch/PR if the user chose that PR strategy.

```markdown
## Phases

### Milestone 1: {Name} — {one-line summary}

| # | Phase |
|---|-------|
| 1 | {Phase name} |
| 2 | {Phase name} |
| 3 | {Phase name} |

### Milestone 2: {Name} — {one-line summary}

| # | Phase |
|---|-------|
| 4 | {Phase name} |
| 5 | {Phase name} |
```

Phase numbering is global (continuous across milestones). Milestones are for human navigation and PR boundaries — they don't change how phases are executed or tracked in state.json.
