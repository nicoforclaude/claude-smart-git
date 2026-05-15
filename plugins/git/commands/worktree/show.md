---
description: "List all worktrees with their branch and pending changes status"
argument-hint: ""
allowed-tools: Bash, Skill(windows-shell:windows-shell)
---

# Git: Show Worktree

List all worktrees for the current repository, showing branch and pending changes for each.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Fetch and list worktrees

```bash
git fetch origin --quiet
git worktree list
```

## Step 3 — Check each worktree

For each worktree path from Step 2, run:

```bash
git -C {worktree-path} branch --show-current
git -C {worktree-path} status --short
git -C {worktree-path} log origin/main..HEAD --oneline 2>/dev/null
```

## Step 4 — Report

Present a table:

```
Worktrees:

  📁 /path/to/worktree  (main worktree)
     Branch:   main
     Changes:  clean
     Ahead:    —

  📁 /path/to/worktree2
     Branch:   nico/feature/something
     Changes:  3 files modified, 1 untracked
     Ahead:    2 commits
```

Mark the current worktree with `← current`.
