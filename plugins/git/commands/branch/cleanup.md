---
description: "Delete a fully merged branch locally and remotely"
argument-hint: "[branch-name] (omit to pick from list)"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: Branch Cleanup

Delete a fully merged branch — locally and from remote. Refuses to delete unmerged branches.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Fetch and identify target branch

```bash
git fetch origin --prune --quiet
```

**If `$ARGUMENTS` provided** — use that as the target branch name, skip the picker.

**If no argument** — list candidates (local branches already merged into main):

```bash
git branch --merged origin/main | grep -v "^\*" | grep -v "main"
```

If the list is empty, report: "No fully merged local branches found. Nothing to clean up." and stop.

If there are candidates, use `AskUserQuestion` to let the user pick which branch to delete.

## Step 3 — Verify the branch is fully merged

```bash
git log origin/main..{branch} --oneline
```

If this returns any commits — the branch is **not fully merged**. Stop and report:

> "⚠️ Branch `{branch}` has N commit(s) not yet in main. Cleanup refused to prevent data loss."

If empty — confirmed merged. Proceed.

## Step 4 — Show plan and ask confirmation

```
🗑️  Branch Cleanup Plan:

Branch:  {branch-name}
Status:  Fully merged into main ✅

  1. Switch to main          git checkout main
  2. Pull latest             git pull origin main
  3. Delete local branch     git branch -d {branch}
  4. Delete remote branch    git push origin --delete {branch}

Shall I proceed?
```

Use `AskUserQuestion`:
- "Yes, delete it"
- "Cancel"

## Step 5 — Execute

```bash
git checkout main
git pull origin main
git branch -d {branch}
git push origin --delete {branch}
```

Report each step as it completes. If remote delete fails (already deleted), note it and continue — not an error.

Final report: "Branch `{branch}` deleted locally and from remote."
