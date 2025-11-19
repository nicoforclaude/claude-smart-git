---
name: changes-analyzer
description: Analyzes git changes and recommends atomic commit strategies with readiness assessment (✅/🚧/⚠️/🗑️). Use for multi-commit workflows and working tree review (including startup check).
---

# Git Changes Analyzer Skill

This skill defines the logic of processing git repository changes to recommend logical grouping into atomic commits, along with readiness assessment.

## Core Steps of Analysis

1. **Analyze Changes**: Understand what changed in the working tree
2. **Group Logically**: Group related changes by topic, type, and scope
3. **Assess Readiness**: Judge if changes are ready to commit or still WIP
4. **Recommend Strategy**: Suggest how to split into atomic commits
5. **Generate Messages**: Draft commit messages following conventions

## Analysis Process

### Prerequisites

You will be provided with git command outputs:
- `git status` - All changes (staged and unstaged)
- `git diff --stat` - File-level statistics
- `git diff` - Full diff for readiness assessment

### Step 1: Classify Changes

For each changed file, determine:

**Change Type:**
- `feat` - New feature or functionality
- `fix` - Bug fix
- `refactor` - Code restructuring without behavior change
- `docs` - User-facing documentation only (README, user guides, tutorials)
- `claude` - Claude Code tooling (`.claude/` - commands, agents, skills, settings)
  - **IMPORTANT**: `.md` files under `.claude/` are scripts/config, NOT docs - use `claude` type
- `style` - Formatting, whitespace
- `test` - Test files
- `chore` - Build, config, dependencies
- `perf` - Performance improvements

**Classification Priority:**
1. Check file path first: If under `.claude/` directory → use `claude` type
2. Then check file extension and content for other types
3. Use `docs` ONLY for user-facing documentation outside `.claude/`

**Scope/Module:**
- Which component, package, or subsystem is affected
- Entry points vs supporting files

**Topic/Feature:**
- What high-level goal does this change serve
- Group related files by common purpose

### Step 3: Assess Readiness

For each group, judge readiness level:

- ✅ **Ready to commit**: Changes are complete, coherent, and functional
  - No TODO/FIXME comments added
  - No debug logging or commented code
  - Follows project conventions
  - All related changes present (no half-implemented features)

- 🚧 **Work in progress**: Incomplete or experimental
  - Contains TODO/FIXME/WIP markers
  - Debug code or console.log statements
  - Incomplete implementation
  - Missing tests or documentation for new features

- ⚠️ **Needs review**: Unclear or mixed concerns
  - Multiple unrelated changes in same files
  - Unclear intent or purpose
  - Mix of refactor + feature
  - Potential breaking changes

- 🗑️ **Should remove**: Accidental or unwanted changes
  - Temporary files (nul, .DS_Store, etc.)
  - Debug artifacts
  - Unintended whitespace changes
  - Commented-out code

### Step 3: Identify Dependencies

Determine commit order:
- Changes that others depend on must come first
- Independent changes can be committed in any order
- Flag circular dependencies

### Step 4: Recommend Commit Strategy

Group files into logical commits:
- Each commit should be atomic (one purpose)
- Each commit should be complete (tests pass)
- Related changes stay together
- Unrelated changes are separated

## Output Format

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

## Readiness Assessment Guidelines

When analyzing diff content, look for:

**✅ Ready indicators:**
- Clean, purposeful changes
- Consistent style and formatting
- Complete implementations
- Proper error handling
- Updated tests and docs

**🚧 WIP indicators:**
- `TODO`, `FIXME`, `WIP`, `XXX`, `HACK` comments
- `console.log`, `print`, `debugger` statements
- Commented-out code blocks
- Incomplete function implementations (throw new Error("Not implemented"))
- Missing error handling

**⚠️ Needs Review indicators:**
- Large refactors mixed with features
- Changes to many unrelated files
- Unusual patterns or approaches
- Breaking changes without migration path

**🗑️ Should Remove indicators:**
- Files named `nul`, `undefined`, `null`
- System files (`.DS_Store`, `Thumbs.db`)
- Editor backup files (`*.swp`, `*~`)
- Large debug output files

## Integration with Commands

This skill is invoked by `/git_prepare_commits` command which provides git status and diff output.

## Additional Resources

See `examples.md` and `commit-patterns.md` for detailed patterns and scenarios.

## Important Notes

- **Never auto-commit** - Only analyze and recommend
- **Respect atomic commits** - Don't group unrelated changes
- **Consider reviewability** - Smaller, focused commits are better
- **Follow project conventions** - Match existing commit message style
- **Be cautious with WIP** - Clearly flag incomplete work

