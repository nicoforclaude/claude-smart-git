---
description: "Delete the current branch once fully merged into main — lands on main when done"
argument-hint: "[branch-name] (omit to use current branch)"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: Branch Cleanup

Delete the current branch (or a named branch) after it has been fully merged into main. Refuses unmerged branches and protected branches. Lands on main when done.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Read state (parallel)

Run these in parallel:

```bash
# Current branch + remote state
git branch --show-current
git fetch origin --prune --quiet
```

```bash
# Branch convention doc (same search as branch:create)
find . -maxdepth 4 -name "branch*.md" -o -name "*branch-naming*" -o -name "*conventions*" 2>/dev/null | grep -v ".git/"
```

If a convention doc is found, read it and extract any protected branch patterns.

Default protected patterns (always applied, regardless of convention doc):

- `deploy/*`
- `release/*`
- `staging`
- `production`
- `main`
- `master`
- `develop`

## Step 3 — Determine target branch

**If `$ARGUMENTS` provided** — use that as the target branch.

**If no argument** — use the current branch from Step 2.

If the target matches a **protected pattern** — stop and report:

> "⛔ Branch `{branch}` matches a protected pattern (`{pattern}`). Cleanup skipped."

## Step 4 — Verify the branch is fully merged

```bash
git log origin/main..{branch} --oneline
```

If this returns any commits — the branch is **not fully merged**. Stop and report:

> "⚠️ Branch `{branch}` has N unmerged commit(s). Not ready for cleanup."
>
> "Tip: run `/git:branch:cleanup:scan` to find branches that are already merged and ready to go."

If empty — confirmed merged. Proceed.

## Step 5 — Check for pending changes

```bash
git status --short
```

If there are any staged or unstaged changes — stop and report:

> "⚠️ You have pending changes. Please commit or stash them before running cleanup."

If working tree is clean — proceed.

## Step 6 — Show plan and ask confirmation

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

## Step 7 — Execute

```bash
git checkout main
git pull origin main
git branch -d {branch}
git push origin --delete {branch}
```

Report each step as it completes. If remote delete fails (already deleted remotely), note it and continue — not an error.

Final report:

> "Branch `{branch}` deleted locally and from remote. Now on `main`. ✅"
