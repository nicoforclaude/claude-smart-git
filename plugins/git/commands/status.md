---
description: "Show git status — current branch, staged, unstaged, and untracked files"
argument-hint: ""
allowed-tools: Bash, Skill(windows-shell:windows-shell)
---

# Git: Status

Run `git status` and present the output clearly.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Run

```bash
git branch --show-current
git status
```

Show the output as-is. No summarizing, no filtering — full git status.
