---
description: "Quick git status check"
allowed-tools: Bash(pwd:*), Bash(git status:*), Bash(git rev-parse:*)
---

! pwd

! git status



## Brief information

First, show the working directory to the user.

For git status:
- If working directory is the root commander location (e.g., your workspace root), navigate to .claude folder first: `cd .claude && git status`
- For child repos, just use the git status result as is

You should say either:
- working tree clean
- changes stats ( 🟢added, 📝 modified, 🗑️ deleted) totall both across staged and unstaged - no difference here
  - in this case you suggest, "Tell me if I should list files (list, yes)"
  - if user does not catch up your suggestion, you exit the command context, it's done
  - quick summary on topics in changes

**Topics summary guidelines:**
- Be **specific and short** - find the most specific commonizer for the changes (including per-line changes if number of changed files is small)
- Bad: "README file documentation update"
- Good: "README update: new clock feature"
- Bad: "claude commands" (too broad)
- Good: "commit workflow refactor" (specific)
- Bad: "documentation updates" (too broad)
- Good: "editor brevity metrics" (specific)
- If changes span multiple specific topics, list them: "commit workflow, linter setup"
- Avoid generic terms when specific ones exist

Do not differ between staged and unstaged. This one is wrong:
```text
 7 changes total:
  - 1 staged (new file)
  - 6 unstaged (4 deleted, 2 modified)
```

## In case requested more information with "list files"

Please output the result nicely with icons and paths like here:````

```text
  Staged changes:
  - ✅ New file: .claude/commands/startup_check_full.md

  Unstaged changes:
  - 🗑️ Deleted: .claude/agents/git-agent.md
  - 🗑️ Deleted: .claude/agents/linter-agent.md
  - 📝 Modified: .claude/agents/startup-agent.md
  - 🗑️ Deleted: .claude/commands/git_main_catchup.md
  - 🗑️ Deleted: .claude/commands/git_prepare_commit.md
  - 🗑️ Deleted: .claude/commands/startup_check.md
  - 🗑️ Deleted: .claude/commands/startup_check_full.md (staged as new, but deleted in working tree)
  - 📝 Modified: .claude/settings.local.json
```

Also, provide extra information from git status you find relevant for user before starting work.