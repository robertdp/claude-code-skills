# claude-code-skills

A [Claude Code plugin](https://docs.anthropic.com/en/docs/claude-code/plugins) with engineering workflow skills for planning, review, quality, and tooling.

## Installation

```bash
# Test locally
claude --plugin-dir /path/to/claude-code-skills

# Or install from GitHub
claude plugin add robertdp/claude-code-skills
```

Skills are invoked as `/robert:<skill-name>` (plugin) or `/<skill-name>` (standalone).

## Skills

### Planning & Design

**interview** — Structured interviews to gather requirements, surface unknowns, and explore tradeoffs before planning or decision-making. Adaptive questioning across 2–3 rounds, with autonomous codebase and web research between rounds.

**phase** — Break a spec or requirements into a phased implementation plan. Generates self-contained plan files (PLAN.md, phase files, state.json) that any agent can pick up and execute with minimal context. Includes adversarial plan review and coverage validation.

**design** — Research, design, and plan before building. Two modes: greenfield (problem → research → options → validated plan) and review (stress-test an existing design). Uses subagents for independent research and critical validation.

### Review & Quality

**improve** — Multi-frame adversarial critique with guided issue resolution. Selects from 12 reviewer frames (Pessimist, Simplifier, Engineer, Adversary, etc.) based on target type, then walks through each finding one at a time with solution options and tradeoffs.

**simplify-changes** — Analyse a PR or local branch diff for simplification opportunities: duplication, unnecessary complexity, dead code, verbose patterns, and redundant operations. Works with both `gh pr` and local git diffs.

**harden-instructions** — Audit CLAUDE.md, rules, and instruction files against research-backed LLM compliance principles. Checks positioning, atomicity, soft language, conflicts, specificity, and identifies hook candidates. Produces a filtered report and optional rewrite.

### Tooling

**bootstrap** — Build Claude Code skills, custom subagents, plugins, and agent team configurations. Interviews for requirements, then generates the full file structure with appropriate enforcement mechanisms.

**worktree** — Give every task its own isolated working directory via git worktrees. Conventions for branch naming, directory layout, and cleanup.

**cmux** — Control the [cmux](https://cmux.dev) terminal multiplexer. Manage workspaces, terminals, browser automation, sidebar status, and notifications via the cmux CLI.

## License

MIT
