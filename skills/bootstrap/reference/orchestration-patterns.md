# Orchestration Patterns

Topology patterns for multi-agent systems. Select based on task structure, not complexity.

## Hub-and-Spoke

One coordinator delegates to focused specialists, then synthesises results.

```
         ┌─── Specialist A ───┐
         │                    │
Main ────┼─── Specialist B ───┼──── Synthesis
         │                    │
         └─── Specialist C ───┘
```

**When to use**: Multiple independent analyses of the same input. Each specialist examines one dimension.

**Key patterns**:
- Single discovery pass first — build one authoritative map (codebase, data, context)
- Inject shared map into all specialists to avoid redundant exploration
- Launch all specialists as **foreground Agent calls in a single message** for parallel execution — do NOT use `run_in_background` (see [decision-frameworks.md](decision-frameworks.md#parallel-execution-vs-background-mode))
- Scope budgets per specialist (primary: 10-15 findings, secondary: 5-8) to force prioritisation
- **The orchestrator is an error absorption checkpoint.** Cross-checking specialist outputs against each other and against the shared map catches errors that independent architectures miss. This verification bottleneck is a feature — don't bypass it.
- Synthesis phase: deduplicate across specialists, resolve conflicts conservatively, identify coverage gaps
- Compound analysis: find interactions only visible when all dimensions are combined

**Communication contract**: Each specialist returns structured findings with file:line references and coverage report (what was examined vs skipped).

**Example**: Code health analysis — discover codebase → parallel analysis (structure, security, reliability, infrastructure) → synthesise → compound risk chains.

## Fan-Out / Fan-In

Single input, parallel processing, merged output. Similar to hub-and-spoke but workers do the same type of work on different partitions.

```
         ┌─── Worker (partition 1) ───┐
Input ───┼─── Worker (partition 2) ───┼─── Merge
         └─── Worker (partition 3) ───┘
```

**When to use**: Same operation applied to many items. Test suites, batch file processing, multi-repo analysis.

**Key patterns**:
- Partition input before fanning out
- Each worker gets identical instructions but different data
- Merge step combines results and resolves conflicts
- Workers run on cheaper models (Haiku/Sonnet) for cost efficiency
- **Verification is essential, not optional.** Independent agents without a verification bottleneck amplify errors — each agent's mistakes propagate unchecked into the merge. Always validate individual outputs before combining.

**Communication contract**: Workers return structured results in a consistent format. Merge step expects uniform shape.

## Pipeline

Sequential phases, each producing a document consumed by the next.

```
Phase 1 ──doc──► Phase 2 ──doc──► Phase 3 ──doc──► Phase 4
```

**When to use**: Each phase requires the output of the previous phase. Scope → design → plan → build.

**Key patterns**:
- Documents are the interface — each phase reads input from disk, writes output to disk
- Context clearing between long-running phases prevents degradation
- Quality gates at each transition: output validated before the next phase starts
- Checkpoint state on disk enables mid-session pauses and cross-session resumption
- The planner cannot validate its own plan — separate validation context

**Context clearing strategy**:
- **Early phases** (scope → design → plan): Continue in same session. Codebase context carries forward usefully.
- **Execution phases** (plan → build): Clear context. Build orchestrator is long-running and must start fresh.

**Communication contract**: Documents on disk. Each phase reads its predecessors' documents. No implicit state through conversation history.

## Router

Single entry point dispatching to different workflows based on intent.

```
           ┌──► Workflow A
Intake ────┼──► Workflow B
           └──► Workflow C
```

**When to use**: One skill handles 3+ distinct intents with shared principles.

**Key patterns**:
- Essential principles loaded for ALL routes (always in context)
- Specific workflow loaded only for the selected route (progressive disclosure)
- Each workflow is self-contained with its own required reading, process, and success criteria
- New workflows can be added without changing existing ones

**Communication contract**: Intake extracts intent. Router loads the relevant workflow file. Workflow operates independently.

## Hybrid

Combinations of the above. Common: pipeline with fan-out at one stage.

```
Phase 1 ──doc──► Fan-Out ──► Merge ──doc──► Phase 3
                  ├─► Worker A
                  ├─► Worker B
                  └─► Worker C
```

**When to use**: Complex systems where different stages have different structures.

**Design principle**: Each stage uses the simplest pattern that fits. Don't force everything into one topology.

## Communication Contracts

Define what each agent receives and returns. This is the most important design decision.

### Input contract
- What context the agent needs (working directory, file paths, codebase map, prior phase output)
- Format: explicit paths, not implicit conversation state
- Subagents do NOT inherit the orchestrator's working directory — pass it explicitly

### Output contract
- Summary size: 1-2k tokens for results returned to main agent
- Structured format: consistent shape the consumer can parse
- Coverage: what was examined and what was skipped
- Confidence: flag uncertain findings explicitly

### Intermediate files over inline output

When one subagent's output feeds another subagent, write to a file instead of passing through the orchestrator. The orchestrator passes file paths, not content.

**Bad** — orchestrator holds all outputs in context:
```
Orchestrator launches Agent A → receives full output
Orchestrator launches Agent B → passes Agent A's output verbatim in prompt
```

**Good** — orchestrator holds only file paths:
```
Orchestrator launches Agent A → writes to file, returns file path
Orchestrator launches Agent B → receives file path, reads from disk
```

This matters because:
- The orchestrator's context stays lean across long-running workflows
- Subagent outputs can be arbitrarily large without affecting the orchestrator
- Files survive session boundaries — if the orchestrator crashes, the intermediate work isn't lost
- Files are inspectable — the user can read them directly

**When to use**: Any time subagent output exceeds a short summary (more than ~500 tokens), or when the output feeds a downstream subagent. The producing subagent writes to disk and returns only the file path and a short summary for display.

**Where to write**: Use a temporary working directory for intermediate files that aren't meaningful artefacts (investigation briefs, individual reviewer findings before synthesis, partial results). Create a session-scoped directory under `/tmp` using `CLAUDE_SESSION_ID` for uniqueness.

**IMPORTANT**: `${CLAUDE_SESSION_ID}` is a string substitution that only works in SKILL.md content — it gets replaced before Claude sees the text, like `$ARGUMENTS`. It is **not** an environment variable and is **not** available in workflow files, subagent prompts, or Bash commands.

**Pattern**: Define the resolved path in SKILL.md where the substitution happens, then reference it from workflow files and pass the resolved absolute path to subagents.

In SKILL.md:
```markdown
### Working Directory
The session working directory for intermediate files is: `/tmp/claude-${CLAUDE_SESSION_ID}/my-skill`
```

In workflow files or subagent prompts, reference the resolved path from SKILL.md — never use `${CLAUDE_SESSION_ID}` directly.

Reserve project directories (e.g., `blueprints/`) for final outputs that the user or downstream phases need to read. Intermediate files in `/tmp` are automatically cleaned up on reboot and don't clutter the project.

### max_turns safety nets

Always set `max_turns` on subagent calls. A stuck agent that loops until context exhaustion wastes far more resources than a generous limit. Err on the high side — failing just before success is worse than slightly overusing context.

Rules of thumb:
- Implementation agents (complex, multi-file work): 50–60
- Investigation/research agents: 30–40
- Review/critique agents: 25–30
- Synthesis/merge agents: 25–30
- Simple single-purpose agents (PR creation, formatting): 20

These are circuit breakers for stuck agents, not expected turn counts. Set them high enough that a healthy agent never hits them.

### Passing context to subagents
Always include in the Agent prompt:
- Working directory (absolute path)
- Branch name (if git operations needed)
- Absolute paths to documents the subagent needs
- Any shared artefacts (codebase map, prior findings)

## Context Strategy

### When to keep shared context
- Early phases of a pipeline where prior context is still fresh and relevant
- Rapid back-and-forth refinement
- Tasks that build directly on conversation state

### When to clear context
- Before long-running execution phases
- When context has accumulated irrelevant information
- Between independent tasks in the same session

### When to fork (isolated subagent)
- Self-contained work that benefits from no noise
- Parallel tasks that shouldn't interfere with each other
- Verbose operations that would bloat the main context
- Tasks requiring specific tool restrictions

## Failure Handling

### Subagent returns garbage
1. Check if the input contract was violated (missing context, wrong paths)
2. If input was correct: retry once with more explicit instructions
3. If retry fails: escalate to the user with what was tried and what failed

### Subagent times out
1. Check maxTurns setting — was it too low?
2. Check if the task scope was too broad for one agent
3. Consider partitioning the work (fan-out)

### Validate parallel outputs before merging

After parallel subagents return, the orchestrator must verify each output (file exists, non-empty, coherent) before launching synthesis or merge. If a subagent failed, retry it once — but only if the retry is idempotent. A subagent that produced no side effects (no files written, no commits) is safe to retry. A subagent that wrote partial output to a known, dedicated output file (e.g., a reviewer's findings file) can be retried after removing that specific file. A subagent that committed changes, modified shared files, or produced side effects beyond its designated output is **not safe to retry** — stop and report to the user. Do not autonomously clean up side effects you aren't certain about; partial work may be valuable. If the retry fails or cleanup isn't safe, the decision depends on the topology:

**Hub-and-spoke** (each specialist covers a distinct dimension): Do not proceed with partial coverage. Each specialist was selected because its perspective matters — a missing dimension degrades the synthesis. Retry the failed specialist once. If the retry also fails, stop and report the failure.

**Fan-out** (same operation on different partitions): Partial results are acceptable. N-1 of N partitions still have value. Flag the gap in the merge output and continue.

