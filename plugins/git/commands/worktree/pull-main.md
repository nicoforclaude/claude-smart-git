---
description: "Fetch origin/main and fast-forward all clean worktrees"
argument-hint: ""
allowed-tools: Bash, Skill(windows-shell:windows-shell)
---

# Git: Worktree — Pull Main to All

Fetch `origin/main` once, then fast-forward every worktree that is clean and hasn't diverged.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Fetch

```bash
git fetch origin main
```

## Step 3 — Discover worktrees

```bash
git worktree list
```

Extract the path (first field) from each line.

## Step 4 — Check and update each worktree in parallel

For each worktree path, run in parallel:

```bash
git -C "<path>" status --porcelain
git -C "<path>" rev-parse HEAD
git -C "<path>" rev-parse origin/main
git -C "<path>" merge-base --is-ancestor HEAD origin/main
```

Decision logic per worktree:

| Condition | Action | Status |
|---|---|---|
| `status --porcelain` has output | Skip | ⚠️ Skipped — dirty |
| HEAD == origin/main | Skip | ✅ Already up to date |
| `merge-base --is-ancestor` exits 0 | Run `git -C "<path>" merge --ff-only origin/main` | ✅ Fast-forwarded |
| `merge-base --is-ancestor` exits 1 | Skip | ⚠️ Skipped — diverged |

## Step 5 — Report

Print a summary table:

```
┌──────────────────────────┬──────────────────────┬────────────────────────────────┐
│ Worktree                 │ Branch               │ Result                         │
├──────────────────────────┼──────────────────────┼────────────────────────────────┤
│ main                     │ main                 │ ✅ Fast-forwarded (3 commits)   │
│ feature/auth             │ nico/feature/auth    │ ⚠️  Skipped — dirty             │
│ hotfix                   │ nico/hotfix/login    │ ⚠️  Skipped — diverged          │
└──────────────────────────┴──────────────────────┴────────────────────────────────┘
```

If any worktree errored, print the full error output below the table.
