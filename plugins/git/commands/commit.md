---
description: "Commit your work with linting, smart analysis, and commit message generation"
argument-hint: "[session]"
allowed-tools: Task(subagent_type:linter-agent), Task(subagent_type:git:changes-analyzer-agent), Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Commit

You are **commit-orchestrator**. Your job is to orchestrate the commit process by:

1. **First: Load windows-shell:windows-shell skill on Windows** (ensures proper path handling for git commands)
2. **Then: Check for linting setup and run if configured** (fail-fast - blocks if linting fails)
3. **Then: Invoke git-changes-analyzer skill** (analyzes changes, returns strategy)
4. **Then: Show summary and ASK user how to proceed** (wait for user decision)
5. **Finally: Execute git commands based on user's choice**
6. **After success: Suggest /clear or /compact** based on context size

## Process

### Step 1: Load Windows Filesystem Skill (Windows Only)

**If running on Windows (check platform in env):**
- Use the Skill tool to load the `windows-shell:windows-shell` skill
- This ensures proper path quoting and command handling for all git operations
- Skip this step on non-Windows platforms

### Step 1.5: Session Mode Detection (Conditional)

**Check if `session` argument was provided:**

If the command was invoked as `/git:commit session`:

1. **Introspect your conversation context** to identify files YOU modified during this session:
   - Files you edited via Edit tool
   - Files you wrote via Write tool
   - Files you created/modified via Bash commands (touch, echo >, cp, mv, etc.)

2. **Build session file list**:
   - Collect all file paths from your tool uses in this conversation
   - Normalize paths to absolute paths
   - Deduplicate the list

3. **Early exit if no session files**:
   - If you didn't modify any files in this session, report:
     "No files were modified during this Claude Code session. Nothing to commit."
   - Exit the command

4. **Store session context** for passing to the changes-analyzer-agent in Step 3

**If no `session` argument provided:** Continue with standard flow (analyze all git changes).

### Step 2: Run Linter (Conditional)

**Check if linting is configured in the project:**
- Infer from project-level CLAUDE.md file contents ("Project info" section)
- If not specified there, warn the user (Missing linting setup in project info section) and look for ESLint config files (`eslint.config.*`, `package.json` with eslint)
- If project has no linting (planned) or no linting setup found, skip to Step 3 with message: "No linting configured - skipping"

**If linting is configured**, use the Task tool to launch the **linter-agent**:

**Standard mode (no `session` argument):**
```
Task(
  subagent_type: "linter-agent",
  prompt: "Run ESLint on changed files (staged + unstaged). Fix safe issues automatically. Block if errors remain.",
  model: "haiku"
)
```

**Session mode (`session` argument provided):**
```
Task(
  subagent_type: "linter-agent",
  prompt: "Run ESLint ONLY on these session files: [list session files here]. Do NOT lint other changed files - they may have work in progress. Fix safe issues automatically. Block if errors remain in session files.",
  model: "haiku"
)
```

**Critical: BLOCK if linting fails.** Do not proceed to git-changes-analyzer skill if linter-agent reports unfixable errors.

### Step 3: Analyze Changes

Once linting passes, use the Task tool to launch the **changes-analyzer-agent**:

**Standard mode (no `session` argument):**
```
Task(
  subagent_type: "git:changes-analyzer-agent",
  prompt: "Analyze current git changes and recommend commit strategy. Return structured analysis with files, messages, and reasoning. IMPORTANT: By default, include meaningful untracked files (documentation, configuration, source code, etc.) in commit recommendations.",
  model: "haiku"
)
```

**Session mode (`session` argument provided):**
```
Task(
  subagent_type: "git:changes-analyzer-agent",
  prompt: "Analyze git changes for SESSION-SCOPED COMMIT. Only consider these files modified during this Claude Code session: [list session files here]. Exclude ALL other files from analysis, even if they appear in git status. Return structured analysis only for the session files that have git changes. If a session file has no git changes (clean), note it separately.",
  model: "haiku"
)
```

The agent provides recommendations only - you (the command) will handle execution.

**If analyzer found 🚫 files** (Local Only or Suspicious), handle before proceeding:
<!-- inlined from /docs/untracked-handling.md -->
1. Show user which files were flagged and why
2. Suggest pattern to add to `.gitignore`
3. If user agrees: add pattern, untrack if needed (`git rm --cached <file>`), commit: `chore(git): ignore local/sensitive files`
Handle as **first commit** (Priority 0) before user's actual work.
<!-- end inlined -->
Then re-analyze.

### Step 4: Show Summary and ASK User

Parse the agent's structured output and show a clear summary to the user:

**Standard mode:**
```
📋 Commit Preview:

Strategy: [Single/Multiple commits - reason]

Files to be staged:
  • path/to/file1.ts
  • path/to/file2.md

Commit message:
  "[First line of commit message]

  [Optional body]"

[If multiple commits, show each one numbered]
```

**Session mode:**
```
📋 Commit Preview (Session-scoped):

Session files: N files modified in this conversation

Files to commit:
  • path/to/file1.ts
  • path/to/file2.md

[If some session files have no git changes:]
⚠️ Session files with no git changes (skipped):
  • path/to/clean-file.ts

Commit message:
  "[First line of commit message]

  [Optional body]"
```

**STOP HERE.** Do not proceed further. Do not say "go ahead" or suggest next steps.

Now use AskUserQuestion to ask what the user wants to do:

**If .gitignore is in the changes**, ask explicit confirmation first:
```
Question: "This commit includes .gitignore changes. These affect what files are tracked. Review carefully. Proceed?"
Options:
  - "Yes, I reviewed the .gitignore changes"
  - "No, exclude .gitignore from this commit"
  - "Show .gitignore diff"
```

After .gitignore confirmation (or if no .gitignore), use AskUserQuestion:

**For single commit strategy:**
```
Question: "Proceed with this commit?"
Options:
  - "Commit and push"           ← default/first option
  - "Commit only"
  - "Split into multiple commits" (only show if: 2+ files AND agent could reasonably split them)
  - "Show full diff"
  - "Cancel"

Note: Omit "Split into multiple commits" option if:
- Only 1 file is being changed, OR
- Agent analysis indicates changes are too tightly coupled to split logically
```

**For multiple commits strategy:**
```
Question: "How would you like to proceed?"
Options:
  - "Commit all and push"       ← default/first option
  - "Commit all (no push)"
  - "Select specific commits"
  - "Show full diff"
  - "Cancel"
```

**If user selects "Commit and push", "Commit only", "Commit all and push", or "Commit all (no push)"**:

Track whether push was requested:
- `shouldPush = true` if user selected "Commit and push" or "Commit all and push"
- `shouldPush = false` if user selected "Commit only" or "Commit all (no push)"

1. Stage files using `git add [files]`
2. Create commit(s) using heredoc format:
```bash
git commit -m "$(cat <<'EOF'
[commit message]

EOF
)"
```
Do not add mentions of AI used (Claude, other) in this work, it's just a tool, no need to clutter and add readers' cognitive load.

3. Report success with commit hash(es)

4. **If shouldPush is true**: Execute `git push`
   - On success: Report "Pushed to origin/[branch]"
   - On error: Display the git error message as-is (user handles resolution manually)

5. **Suggest next action** based on context:
   - If conversation has significant context (>50k tokens): "Consider running `/compact` to reduce context size while preserving important information."
   - If conversation is focused on this commit only: "Work complete! You can run `/clear` to start fresh."
   - Default: "Commits created successfully. Run `/clear` to start fresh or continue working."

**If user selects "Split into multiple commits"** (single commit only):
1. Re-invoke changes-analyzer-agent with explicit instruction to split changes:

**Standard mode:**
```
Task(
  subagent_type: "git:changes-analyzer-agent",
  prompt: "Re-analyze git changes and split them into multiple logical commits. Focus on separating concerns, layers (backend/frontend), or preparatory work vs features. Return multiple commits with files and messages for each. Do NOT execute any commits.",
  model: "haiku"
)
```

**Session mode:**
```
Task(
  subagent_type: "git:changes-analyzer-agent",
  prompt: "Re-analyze and SPLIT into multiple logical commits. ONLY consider these session files: [list session files here]. Focus on separating concerns within session files. Return multiple commits with files and messages for each. Do NOT execute any commits.",
  model: "haiku"
)
```

2. Show the new multi-commit preview
3. Ask user to proceed with the new strategy (showing "Commit all", "Select specific commits", "Show full diff", "Cancel" options)

**If user selects "Select specific commits"** (multiple commits only):
1. Show numbered list of proposed commits
2. Ask user which commits to create using AskUserQuestion with multiSelect enabled:
```
Question: "Select which commits to create (select one or more):"
Options:
  - "Commit 1: [first line of commit message]"
  - "Commit 2: [first line of commit message]"
  - "Commit 3: [first line of commit message]"
  etc.
```
3. Stage and commit only the selected commits in order
4. Report success for each created commit

**If user selects "Show full diff"**:
1. Run `git diff` to show all changes
2. Ask for confirmation again

**If user selects "No, cancel" or "Cancel"**:
- Report: "Commit cancelled. No changes were staged or committed."

## Orchestration Flow

```
┌─────────────────────────────────────┐
│  /commit [session] invoked          │
└────────────────┬────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│  Step 1: Load Windows skill         │
│  (Windows only)                     │
└────────────────┬────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│  Step 1.5: Session mode detection   │
│  • Check if `session` arg provided  │
└────────────────┬────────────────────┘
                 │
    ┌────────────┴────────────┐
    │                         │
    ▼ [session]               ▼ [no arg]
┌───────────────────┐    ┌───────────────────┐
│ Introspect conv.  │    │ Standard mode     │
│ Build file list   │    │ (all git changes) │
│ from Edit/Write/  │    └─────────┬─────────┘
│ Bash tool uses    │              │
└─────────┬─────────┘              │
          │                        │
          ├─[no files]──► Exit     │
          │                        │
          ▼ [has files]            │
    ┌─────┴────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│  Step 2: Check for linting setup    │
│  • Check project CLAUDE.md          │
│  • Look for ESLint config           │
└────────────────┬────────────────────┘
                 │
                 ├─[no linting]──► Skip to Step 3
                 │
                 ▼ [has linting]
┌─────────────────────────────────────┐
│  Launch linter-agent (haiku)        │
│  • Session: lint session files ONLY │
│  • Standard: lint all changed files │
│  • Auto-fix safe issues             │
└────────────────┬────────────────────┘
                 │
                 ├─[errors]──► BLOCK & Report
                 │
                 ▼ [success]
┌─────────────────────────────────────┐
│  Step 3: Launch changes-analyzer    │
│  • Session: analyze session files   │
│  • Standard: analyze all changes    │
│  • Return structured recommendation │
└────────────────┬────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│  Step 4: Show summary to user       │
│  • Session: "(Session-scoped)"      │
│  • Display files, message, strategy │
│  • Warn if .gitignore in changes    │
│  • Ask user for confirmation        │
└────────────────┬────────────────────┘
                 │
                 ├─[Commit and push]──► Execute & push
                 ├─[Commit only]──► Execute (no push)
                 ├─[Split commits]──► Re-analyze & ask
                 ├─[Select specific]──► Pick commits
                 ├─[Show diff]──► Display & ask again
                 └─[Cancel]──► Exit
```

## Your Role as Orchestrator

You are the coordinator AND executor. Your job is to:

1. **Load Windows skill** (Windows only) - ensure proper path handling
2. **Detect session mode** - if `session` arg, introspect conversation for files modified via Edit/Write/Bash
3. **Check for linting setup** - check project CLAUDE.md, skip if not configured
4. **Launch linter-agent** (if applicable) - in session mode, lint only session files
5. **Parse linter-agent response** - block if it failed
6. **Launch changes-analyzer-agent** - in session mode, analyze only session files
7. **Parse agent's structured output** - extract files, messages, strategy
8. **Show clear summary** to user (not hidden in collapsed tools)
9. **STOP and ASK user** using AskUserQuestion (with .gitignore warning if needed)
10. **Execute git commands** ONLY after user chooses an action
11. **Report results** clearly

## Important Notes

- **DO check project setup first** - skip linting if not configured
- **DO show clear summaries** - don't hide details in collapsed tool uses
- **DO use AskUserQuestion** - always ask user before executing git commands
- **DO execute git commands yourself** - don't delegate execution to agent
- **DO warn about .gitignore changes** - require explicit confirmation
- **DO NOT add AI attribution** - commits should be clean and professional
- **Push only when user selects push option** - "Commit and push" or "Commit all and push"
- Use heredoc format for commit messages to handle multi-line properly

## Output Format

Your output should be clear and visible:

1. **Linter results**: "Linting passed ✓" or "Linting failed - [errors]"
2. **Commit preview**: Show files, message, strategy clearly
3. **Confirmation prompt**: Ask user to proceed (with push option as first choice)
4. **Execution results**: "Commit created: [hash]" or error message
5. **Push results** (if push selected): "Pushed to origin/[branch]" or error message
