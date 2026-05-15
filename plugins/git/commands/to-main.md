---
description: "Switch to main, with a safety check for pending changes"
argument-hint: ""
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: To Main

Switch to the `main` branch. Checks for pending changes first and asks before discarding context.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Check current state

```bash
git branch --show-current
git status --short
```

**If already on `main`** — report "Already on main." and stop.

## Step 3 — Handle pending changes

**If working tree is clean** — switch directly:

```bash
git checkout main
git pull origin main
```

Report: "Switched to main and pulled latest."

**If there are pending changes** — show a summary and ask:

```
⚠️  You have pending changes:

  [list of staged/unstaged/untracked files]

What would you like to do before switching?
```

Use `AskUserQuestion`:
- "Stash changes, then switch to main"
- "Leave changes (they'll carry over if untracked)"
- "Cancel — stay on this branch"

**If "Stash":**
```bash
git stash push -m "auto-stash before switching to main"
git checkout main
git pull origin main
```
Report: "Stashed changes, switched to main, pulled latest. Run `git stash pop` to restore."

**If "Leave changes":**
```bash
git checkout main
git pull origin main
```
Report: "Switched to main (changes carried over)."

**If "Cancel":** Report: "Staying on `{branch}`."
