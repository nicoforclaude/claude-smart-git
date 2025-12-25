---
description: "Diagnose and sync branches downstream (deploy→main→working)"
argument-hint: ""
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Sync

Analyze all branch relationships and sync downstream (deploy → main → working).

Uses **matrix approach**: regardless of current branch, shows full status of all branch relationships and offers actions to sync.

## Process

### Step 1: Load Windows Shell Skill (Windows Only)

**If running on Windows (check platform in env):**
- Use the Skill tool to load the `windows-shell:windows-shell` skill
- This ensures proper path quoting and command handling for all git operations
- Skip this step on non-Windows platforms

### Step 2: Pre-checks

1. Check for uncommitted changes:
   ```bash
   git status --porcelain
   ```
   - If output is not empty, STOP and report:
     "Working tree has uncommitted changes. Please commit or stash before syncing."
   - Exit without proceeding

2. Fetch all remotes:
   ```bash
   git fetch --all --prune
   ```

3. Record current branch for later:
   ```bash
   git branch --show-current
   ```

### Step 3: Diagnose (Matrix Approach)

Build a complete picture of all branch relationships.

#### Step 3a: Identify key branches

1. **Current branch**: already recorded from Step 2
2. **Main branch**: check if `main` exists, otherwise `master`
   ```bash
   git rev-parse --verify main 2>/dev/null && echo "main" || echo "master"
   ```
3. **Deploy branches**: find all branches matching deploy patterns
   ```bash
   git branch -a | grep -E "(dev-preview|prod|production)" | sed 's/^[* ]*//' | sed 's|remotes/origin/||' | sort -u
   ```
4. **Working branch**: if current is not main and not a deploy branch, it's a working branch

#### Step 3b: Build sync matrix

Check ALL relevant relationships. For each pair, count commits behind:

| Relationship | Check Command | Condition |
|--------------|---------------|-----------|
| Current ← remote | `git rev-list --count HEAD..@{upstream} 2>/dev/null` | If has tracking branch |
| Main ← remote | `git rev-list --count {main}..origin/{main}` | Always check |
| Main ← deploy branches | `git rev-list --count {main}..{deploy-branch}` | For each deploy branch |
| Working ← main | `git rev-list --count {working}..{main}` | If on working branch |

**For each pair with count > 0, add to findings list.**

### Step 4: Report

Show the sync status matrix to user:

```
📊 Sync Status Matrix

┌─────────────────────────────────────────────────────┐
│ Branch                    ← Source           Behind │
├─────────────────────────────────────────────────────┤
│ webapp-dev-preview        ← remote         2 commits│
│ main                      ← webapp-dev-preview    5 │
│ nico-working-2            ← main          12 commits│
└─────────────────────────────────────────────────────┘
```

**If no items behind:**
- Report: "Everything is synced! All branches are up to date."
- Exit

### Step 5: Suggest Actions

Use AskUserQuestion with multiSelect:true to let user choose which syncs to perform.

Build options dynamically from findings:
- Each finding becomes an option
- Label format: `Pull/Merge {source} → {target} ({N} commits)`
- Description: brief explanation of what will happen

Example:
```
Question: "Select which branches to sync:"
Options:
  - "Pull remote → webapp-dev-preview (2 commits)"
    Description: "Update local webapp-dev-preview from origin"
  - "Merge webapp-dev-preview → main (5 commits)"
    Description: "Bring deploy fixes into main branch"
  - "Merge main → nico-working-2 (12 commits)"
    Description: "Update working branch with latest main"
```

**If user selects nothing or cancels:**
- Report: "Sync cancelled."
- Exit

### Step 6: Preview Selected Actions

For each selected action, show what will be merged:

```
📋 Preview: Merge webapp-dev-preview → main (5 commits)

  • abc1234 Fix production bug in auth
  • def5678 Update API rate limits
  • ghi9012 Hotfix for checkout flow
  • jkl3456 Emergency DB migration
  • mno7890 Revert broken feature flag
```

Use: `git log --oneline {target}..{source}`

### Step 7: Execute Selected Actions

**Order actions by dependency:**
1. Remote pulls first (for all branches that need them)
2. Deploy → main merges
3. Main → working merges

**For each action:**

1. If target ≠ current branch:
   ```bash
   git checkout {target}
   ```

2. Execute the sync:
   - **For remote pulls:**
     ```bash
     git pull --ff-only
     ```
     If fails (not fast-forward), report error and ask user what to do

   - **For local merges:**
     ```bash
     git merge {source} --no-edit
     ```
     If conflicts occur, report and let user resolve manually

3. Report success or failure for each action

4. **On failure:** Ask user whether to continue with remaining actions or stop

### Step 8: Cleanup

1. Return to original branch:
   ```bash
   git checkout {original-branch}
   ```

2. Show summary:
   ```
   ✅ Sync Complete

   Completed:
     • Pulled remote → webapp-dev-preview (2 commits)
     • Merged webapp-dev-preview → main (5 commits)
     • Merged main → nico-working-2 (12 commits)

   You are now on: nico-working-2
   ```

## Edge Cases

### No sync needed
All branches are up to date. Report and exit early.

### Merge conflict
Stop the current merge, report which files conflict, and exit. Let user resolve manually.

### No tracking branch
If a branch has no upstream tracking, skip remote sync check for that branch. Note this in the report.

### Multiple deploy branches
Show all as separate options. User can select which ones to sync.

### Fast-forward not possible
For remote pulls, if `--ff-only` fails, report the issue and suggest user resolve manually (may need to merge or rebase).

## Important Notes

- Always fetch before analyzing to ensure accurate state
- Never force operations - let user handle complex situations manually
- Return to original branch even if some operations fail
- This command is for downstream sync only; upstream (working → main → deploy) should be done via PR/merge process
