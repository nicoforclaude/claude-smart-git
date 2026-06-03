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

For each worktree path from Step 2, run in parallel:

```bash
git -C {worktree-path} branch --show-current
git -C {worktree-path} status --short
git -C {worktree-path} rev-list --count origin/main..HEAD 2>/dev/null
git -C {worktree-path} rev-list --count HEAD..origin/main 2>/dev/null
```

## Step 4 — Report

Present a table:

```
┌──────────────────┬───────────────────────┬─────────────────┬───────┬────────┐
│ Worktree         │ Branch                │ Changes         │ Ahead │ Behind │
├──────────────────┼───────────────────────┼─────────────────┼───────┼────────┤
│ main ← current   │ main                  │ clean           │  —    │  —     │
│ auth             │ nico/feature/auth     │ 3 modified      │  2    │  —     │
│ hotfix           │ nico/hotfix/login     │ clean           │  —    │ 🔴 1   │
└──────────────────┴───────────────────────┴─────────────────┴───────┴────────┘
```

- Mark the current worktree with `← current`
- Show `—` when ahead or behind count is 0
- Prefix behind count with 🔴 when non-zero
