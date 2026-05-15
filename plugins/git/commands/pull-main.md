---
description: "Fetch and merge origin/main into the current branch"
argument-hint: ""
allowed-tools: Bash, Skill(windows-shell:windows-shell)
---

# Git: Pull Main

Merge `origin/main` into the current branch. Reports position before and after.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Report starting position

```bash
git branch --show-current
git fetch origin main
git rev-list --count HEAD..origin/main
```

Report:

```
📍 Starting position:
   Branch: {branch}
   Behind origin/main: {N} commits
```

If `N = 0` — report "Already up to date with origin/main." and stop.

## Step 3 — Merge

```bash
git merge origin/main --no-edit
```

If conflicts occur — report which files conflict and stop. Let user resolve manually.

## Step 4 — Report ending position

```bash
git log --oneline origin/main..HEAD
git rev-list --count origin/main..HEAD
```

Use the **behind count from Step 2** as `{N}` (commits integrated from origin/main).
Use the `rev-list --count origin/main..HEAD` result as `{commits-ahead}` (your branch's own commits).

Report:

```
✅ Merged {N} commits from origin/main

📍 Now at:
   Branch: {branch}
   Ahead of main: {commits-ahead} commits
```

If fast-forward: note "Fast-forward."
