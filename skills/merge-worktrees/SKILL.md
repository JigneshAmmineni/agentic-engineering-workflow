---
name: merge-worktrees
description: Merge the named worktrees from .trees/ back into the current branch.
argument-hint: <worktree 1>, <worktree 2>, ...
disable-model-invocation: true
---

$ARGUMENTS

Use git merge to merge in all the worktrees in the .trees folder which were specified in the arguments above.
