---
description: "Delete the current branch once fully merged into main — lands on main or a working branch when done"
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
git config user.email
```

```bash
# Branch convention doc (same search as branch:create)
find . -maxdepth 4 -name "branch*.md" -o -name "*branch-naming*" -o -name "*conventions*" 2>/dev/null | grep -v ".git/"
```

```bash
# All working branches (any user prefix)
git branch --list "*/working-*"
```

If a convention doc is found:
- Read it and extract any protected branch patterns.
- Extract the **owner prefix** for the current user by matching `git config user.email` against the owner table in the doc (e.g. `kolyaevdokimov@gmail.com` → `nico`).

If no convention doc is found — owner prefix cannot be resolved; the working-branch fallback will be skipped (see Step 7).

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

Determine the **landing branch**:
- If `main` is available (not checked out in another worktree) → land on `main`
- Otherwise → land on a `{owner-prefix}/working-*` branch for the **current user** (resolved in Step 2); ask user to pick if multiple exist

```
🗑️  Branch Cleanup Plan:

Branch:  {branch-name}
Status:  Fully merged into main ✅

  1. Delete local branch     git branch -d {branch}
  2. Delete remote branch    git push origin --delete {branch}
  3. Switch to {landing-branch}   git checkout {landing-branch}
  4. Pull latest             git pull origin main

Shall I proceed?
```

Use `AskUserQuestion`:
- "Yes, delete it"
- "Cancel"

## Step 7 — Execute

```bash
git branch -d {branch}
git push origin --delete {branch}
```

If remote delete fails (already deleted remotely), note it and continue — not an error.

Then land on the branch determined in Step 6:

**If landing on `main`:**
```bash
git checkout main
git pull origin main
```

**If `main` is busy in another worktree:**

Filter the working branches from Step 2 to `{owner-prefix}/working-*` for the current user.

If owner prefix could not be resolved (no convention doc) — stop and report:

> "⚠️ `main` is checked out in another worktree. Cannot resolve owner prefix (no branch naming convention doc found). Please switch manually."

If exactly one `{owner-prefix}/working-*` branch exists — use it automatically.
If multiple exist — ask user to pick via `AskUserQuestion`.
If none exist — stop and report:

> "⚠️ `main` is checked out in another worktree and no `{owner-prefix}/working-*` branch found. Please switch manually."

```bash
git checkout {working-branch}
git pull origin main
```

Final report:

> "Branch `{branch}` deleted locally and from remote. Now on `{landing-branch}`. ✅"
