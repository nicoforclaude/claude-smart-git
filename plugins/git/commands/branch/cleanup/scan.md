---
description: "Scan for fully merged branches and bulk-delete them by rule, with one confirmation"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: Branch Cleanup Scan

Find all local branches fully merged into main, drop the protected ones by rule, and delete the rest after a single confirmation. No per-branch picker. Lands on main when done.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Read state (parallel)

Run these in parallel:

```bash
# Fetch and list merged branches
git fetch origin --prune --quiet
git branch --merged origin/main
```

```bash
# Branch convention doc
find . -maxdepth 4 -name "branch*.md" -o -name "*branch-naming*" -o -name "*conventions*" 2>/dev/null | grep -v ".git/"
```

If a convention doc is found, read it and extract any additional protected branch patterns.

## Step 3 — Filter candidates by rule

Protected patterns (always applied):

- `main`, `master`, `develop`
- `deploy/*`, `release/*`, `staging`, `production`
- `{user}/working*` and any `*/working*` — long-lived working branches (e.g. `nico/working-2`)
- Branches marked `*` (current) or `+` (checked out in a worktree) in the `git branch` output — these can't be deleted anyway

Everything else that is fully merged — typically topic branches like `{user}/feat/*`, `{user}/fix/*`, `{user}/chore/*` — goes to the **candidates** list.

If candidates list is empty — report and stop:

> "No merged branches found that are safe to clean up."

## Step 4 — Single confirmation

Show the full plan once — do NOT ask per branch:

```
🗑️  Branch Cleanup Plan:

Delete (fully merged into main):
  - {branch1}
  - {branch2}
  ...

Keep (protected):
  - {branch} — {matched pattern or "checked out in worktree"}
  ...

For each deleted branch: git branch -d + git push origin --delete.
Then: git checkout main && git pull origin main.
```

Use `AskUserQuestion`:
- "Yes, delete all listed"
- "Cancel"

## Step 5 — Execute

For each candidate branch:

```bash
git branch -d {branch}
git push origin --delete {branch}
```

Report each branch as it's deleted. If remote delete fails (already deleted remotely), note it and continue — not an error. If local delete fails with "not fully merged", skip the branch and report it — never force with `-D`.

After all branches are processed:

```bash
git checkout main
git pull origin main
```

Final report:

> "Deleted N branch(es), kept M protected. Now on `main`. ✅"
>
> Deleted: `{branch1}`, `{branch2}`, ...
