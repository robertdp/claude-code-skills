---
name: worktree
description: Give every task its own isolated working directory via git worktrees. Use proactively for ALL new work on git repos before making changes. Invoked via /worktree or /robert:worktree.
---

## Objective

Create isolated git worktrees for safe experimentation and parallel development.

## Conventions

**Location**: `../worktrees/<project>/<sanitised-branch-name>`

```
parent-dir/
├── my-app/                          # Main checkout
└── worktrees/
    └── my-app/
        ├── feature-auth/
        └── fix-bug-123/
```

**Branch prefixes**: `feature/`, `fix/`, `experiment/`, etc.

**Directory naming**: sanitise branch (slashes → dashes): `feature/auth` → `feature-auth`

## Commands

**Create with new branch**:
```bash
git worktree add ../worktrees/<project>/<dir-name> -b <branch-name>
```

**Create from existing branch**:
```bash
git worktree add ../worktrees/<project>/<dir-name> <existing-branch>
```

**List worktrees**:
```bash
git worktree list
```

**Remove worktree** (commit/stash changes first):
```bash
git worktree remove ../worktrees/<project>/<dir-name>
```

**Force remove** (discards uncommitted changes):
```bash
git worktree remove --force ../worktrees/<project>/<dir-name>
```

**Prune stale entries**:
```bash
git worktree prune
```

## Example Workflow

```bash
# Create worktree for experiment
git worktree add ../worktrees/my-app/experiment-refactor -b experiment/refactor
cd ../worktrees/my-app/experiment-refactor

# If successful: merge back
git checkout main && git merge experiment/refactor

# If failed: discard
cd ../my-app
git worktree remove ../worktrees/my-app/experiment-refactor
git branch -D experiment/refactor
```

## Pitfalls

- Don't create inside main repo - creates nested git structures
- Clean up when done - worktrees persist until removed
- Can't checkout same branch twice - use different branch names
