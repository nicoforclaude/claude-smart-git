---
name: changes-analyzer
description: Analyzes git changes and recommends atomic commit strategies with readiness assessment (✅/🚧/⚠️/🗑️/🚫). Use for multi-commit workflows and working tree review (including startup check).
---

# Git Changes Analyzer Skill

Analyzes git working tree and recommends atomic commit groupings with readiness assessment.

## How It Works

### 1. Understanding Changes

Classify each file by examining:
- **Path & extension** → determines change type (feat/fix/refactor/docs/claude/chore/test/style/perf)
- **Scope** → which component/package affected
- **Topic** → what high-level goal this serves

**Classification rules:**
- Files under `.claude/` → use `claude` type (even .md files - they're scripts/config)
- User docs outside `.claude/` → use `docs` type
- Path check takes priority over extension

### 2. Assessing Readiness

Scan diff content for quality signals:

**✅ Ready** - Clean, complete, conventional:
- No TODO/FIXME/WIP markers
- No debug logging (console.log, print, debugger)
- Complete implementations with error handling
- Updated tests and docs
- Follows project conventions

**🚧 WIP** - Incomplete or experimental:
- TODO/FIXME/XXX/HACK comments
- Debug statements or commented code
- Partial implementations (throw "Not implemented")
- Missing tests or docs for new features

**⚠️ Needs Review** - Mixed concerns or unclear:
- Large refactors mixed with features
- Changes across many unrelated files
- Unclear intent
- Breaking changes without migration

**🗑️ Should Remove** - Unwanted artifacts:
- Temp files (nul, .DS_Store, Thumbs.db)
- Editor backups (*.swp, *~)
- Debug output
- Commented-out code

**🚫 Local Only** - See `untracked.md` for patterns

### 3. Grouping Into Commits

**Prioritize topic grouping over file type:**
- Group by **what** (topic/feature: "linting", "root commander", "git operations")
- Not by **form** (file type: skills vs commands vs agents)
- Example: "Remove linting plugin assets" includes agent + skill + commands together

Apply atomic commit principles:
- One purpose per commit
- Related changes stay together (by topic, not by file extension)
- Unrelated changes separated
- Order by dependencies (prerequisites first)

Each commit gets:
- Conventional message ([type]([scope]): [description])
- Readiness assessment
- Priority/dependency status

### 4. Generating Output

Provide structured analysis in this format:

```
📊 Git Changes Analysis

[Total files changed summary]

---

🎯 Recommended Commit Strategy: [N] commits

[For each recommended commit:]

Commit [N]: [Brief description] ([M] files) [Readiness emoji]
  [File list with emoji indicators]
  Type: [conventional commit type]
  Scope: [affected module/component]
  Priority: [N] ([dependency status])
  Readiness: [assessment with reasoning]

  Suggested commit message:
  ```
  [type]([scope]): [description]

  [optional body explaining why/what]
  ```

---

⚠️ Recommendations:
- [Any warnings, suggestions, or notes]
- [Files to remove or changes to reconsider]

---

📋 Next Steps:
[Suggested commands or actions]
```

## Usage

Invoked by `/git:commit` command which provides git status and diff output.

See `examples.md`, `commit-patterns.md`, `untracked.md`, and `untracked-handling.md` for detailed scenarios.

## Important Constraints

- Never auto-commit - only analyze and recommend
- Respect atomic commits - don't group unrelated changes
- Smaller focused commits preferred for reviewability
- Match existing project commit message style
- Clearly flag incomplete work

