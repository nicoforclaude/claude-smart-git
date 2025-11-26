---
description: "Push commits to remote repository"
argument-hint: ""
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Push

Push current branch commits to remote repository.

## Process

### Step 1: Load Windows Shell Skill (Windows Only)

**If running on Windows (check platform in env):**
- Use the Skill tool to load the `windows-shell:windows-shell` skill
- This ensures proper path quoting and command handling for all git operations
- Skip this step on non-Windows platforms

### Step 2: Check Current State

Run these commands to gather information:
1. `git branch --show-current` - get current branch name
2. `git rev-list --count @{upstream}..HEAD 2>/dev/null || echo "no-upstream"` - commits ahead of remote
3. `git log --oneline @{upstream}..HEAD 2>/dev/null` - list commits to be pushed

### Step 3: Show Summary and Ask User

**If no commits to push:**
- Report: "Nothing to push. Your branch is up to date with origin/[branch]."
- Exit

**If no upstream tracking branch:**
- Report: "No upstream branch configured. Run: `git push -u origin [branch]`"
- Exit

**If commits to push:**
Show summary:
```
📤 Push Preview:

Branch: [branch-name]
Remote: origin/[branch-name]
Commits to push: N

  • [hash1] [message1]
  • [hash2] [message2]
  ...
```

Use AskUserQuestion:
```
Question: "Push N commit(s) to origin/[branch]?"
Options:
  - "Yes, push now"
  - "Show commit details"
  - "Cancel"
```

### Step 4: Execute Push

**If user selects "Yes, push now"**:
1. Execute `git push`
2. On success: Report "Pushed N commit(s) to origin/[branch]"
3. On error: Display the git error message as-is (user handles resolution manually)

**If user selects "Show commit details"**:
1. Run `git log --stat @{upstream}..HEAD` to show detailed changes
2. Ask for confirmation again

**If user selects "Cancel"**:
- Report: "Push cancelled."

## Important Notes

- Always show what will be pushed before executing
- Never force push (`--force`) unless user explicitly requests it
- On push error, display the error and let user handle resolution
- This command is for simple pushes; complex scenarios should be handled manually
