---
description: "Scan for fully merged branches and offer multi-select bulk cleanup"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: Branch Cleanup Scan

Find all local branches that are fully merged into main, filter out protected ones, and let the user pick which to delete. Lands on main when done.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Read state (parallel)

Run these in parallel:

```bash
# Fetch and list merged branches
git fetch origin --prune --quiet
git branch --merged origin/main | grep -v "^\*" | grep -v "^\s*main$" | grep -v "^\s*master$"
```

```bash
# Branch convention doc
find . -maxdepth 4 -name "branch*.md" -o -name "*branch-naming*" -o -name "*conventions*" 2>/dev/null | grep -v ".git/"
```

If a convention doc is found, read it and extract any protected branch patterns.

Default protected patterns (always applied):

- `deploy/*`
- `release/*`
- `staging`
- `production`
- `main`
- `master`
- `develop`

## Step 3 — Filter candidates

For each branch in the merged list:

- If it matches a protected pattern → move it to a **skipped** list, note which pattern matched
- Otherwise → add to **candidates** list

If any branches were skipped, report them before showing the picker:

> "ℹ️ Skipped (protected pattern `{pattern}`): `{branch1}`, `{branch2}`, ..."

If candidates list is empty — report and stop:

> "No merged branches found that are safe to clean up."

## Step 4 — Multi-select picker

Show all candidates. Use `AskUserQuestion` with `multiSelect: true`:

- One option per candidate branch
- Label: the branch name
- Description: "Merged into main, safe to delete"

If user selects nothing — stop with "Nothing selected. No changes made."

## Step 5 — Show plan and ask confirmation

```
🗑️  Branch Cleanup Plan:

Branches to delete:
  - {branch1}
  - {branch2}
  ...

  1. For each branch:
     - Delete local:   git branch -d {branch}
     - Delete remote:  git push origin --delete {branch}
  2. Switch to main   git checkout main
  3. Pull latest      git pull origin main

Shall I proceed?
```

Use `AskUserQuestion`:
- "Yes, delete all selected"
- "Cancel"

## Step 6 — Execute

For each selected branch:

```bash
git branch -d {branch}
git push origin --delete {branch}
```

Report each branch as it's deleted. If remote delete fails (already deleted remotely), note it and continue — not an error.

After all branches are processed:

```bash
git checkout main
git pull origin main
```

Final report:

> "Deleted N branch(es). Now on `main`. ✅"
> 
> Deleted: `{branch1}`, `{branch2}`, ...
