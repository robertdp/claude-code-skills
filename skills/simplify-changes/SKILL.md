---
name: simplify-changes
description: Analyze changes in a PR or working branch and suggest simplifications. Use when user says "simplify changes", "simplify PR", "simplify diff", or invokes /simplify-changes or /robert:simplify-changes.
argument-hint: "[pr-number]"
allowed-tools: Bash(git *), Bash(gh *), Read, Grep, Glob
---

<objective>
Analyze a PR or local branch diff and identify opportunities to simplify the code. Focus on duplication, unnecessary complexity, dead code, verbose patterns, and redundant operations.
</objective>

<quick_start>
1. Gather the diff (from PR number or local branch)
2. Analyze for simplification opportunities
3. Present findings by file with before/after examples
</quick_start>

<workflow>
<step name="gather">
<user-input>
$ARGUMENTS
</user-input>

If arguments contain a PR number or URL, use it to fetch the diff. Otherwise, analyze the local branch.

**PR specified:**

```bash
gh pr view <pr-number> --json title,body,baseRefName,headRefName
gh pr diff <pr-number>
gh pr diff <pr-number> --name-only
```

**No PR specified (local branch):**

```bash
DEFAULT_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || (git branch -r | grep -E 'origin/(main|master)$' | head -1 | sed 's@.*origin/@@'))
git diff "$DEFAULT_BRANCH"...HEAD
git diff --name-only "$DEFAULT_BRANCH"...HEAD
```
</step>

<step name="analyze">
Analyze the diff for:

- **Duplication** — repeated code patterns that could be extracted; copy-pasted logic with minor variations
- **Complex logic** — deeply nested conditionals (3+ levels); convoluted loops; unclear boolean expressions
- **Unnecessary abstractions** — over-engineered patterns for simple tasks; premature generalizations; wrapper functions that add no value
- **Dead code** — unused variables or parameters; unreachable branches; commented-out code
- **Verbose patterns** — code that could use simpler language features; manual reimplementations of stdlib functions; overly explicit code where idiomatic shortcuts exist
- **Redundant operations** — repeated computations that could be cached; unnecessary type conversions; redundant null/error checks
</step>

<step name="output">
Organize findings by file. For each suggestion:

1. **File and location**: `path/to/file.ext:line_number`
2. **Issue**: Brief description of what could be simplified
3. **Suggestion**: Concrete recommendation with before/after examples where helpful

If no simplification opportunities are found, confirm the changes look clean and well-written.
</step>
</workflow>

<success_criteria>
- All meaningful simplification opportunities identified
- Findings are specific with file:line references
- Suggestions include concrete before/after examples where helpful
- No false positives — don't flag idiomatic patterns as problems
</success_criteria>
