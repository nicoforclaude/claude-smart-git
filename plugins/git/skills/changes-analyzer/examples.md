# Git Changes Analyzer - Examples

## Example 1: Clean Feature Addition

### Input
```
Modified: src/features/export/csv.ts
Modified: src/features/export/json.ts
New: tests/export.test.ts
Modified: README.md
```

### Output
```
📊 Git Changes Analysis (4 files)

---

🎯 Recommended Commit Strategy: 1 commit

Commit 1: Add data export functionality (4 files) ✅
  📝 src/features/export/csv.ts
  📝 src/features/export/json.ts
  🟢 tests/export.test.ts
  📝 README.md
  Type: feat
  Scope: export
  Priority: 1 (independent)
  Readiness: ✅ Complete - Includes implementation, tests, and documentation

  Suggested commit message:
  ```
  feat(export): add CSV and JSON export functionality

  Implements data export in two formats with comprehensive tests.
  Updated README with usage examples.
  ```
```

## Example 2: Mixed Concerns - Needs Splitting

### Input
```
Modified: src/auth/login.ts (refactored error handling)
Modified: src/api/users.ts (fixed pagination bug)
Modified: src/utils/validation.ts (added email validator)
Modified: tests/api/users.test.ts
```

### Output
```
📊 Git Changes Analysis (4 files)

---

🎯 Recommended Commit Strategy: 3 commits

Commit 1: Refactor auth error handling (1 file) ✅
  📝 src/auth/login.ts
  Type: refactor
  Scope: auth
  Priority: 1 (independent)
  Readiness: ✅ Complete refactor

  Suggested commit message:
  ```
  refactor(auth): improve error handling in login flow
  ```

Commit 2: Fix user pagination bug (2 files) ✅
  📝 src/api/users.ts
  📝 tests/api/users.test.ts
  Type: fix
  Scope: api
  Priority: 1 (independent)
  Readiness: ✅ Bug fix with test coverage

  Suggested commit message:
  ```
  fix(api): correct pagination logic in users endpoint

  Previously returned incorrect page offsets when page size exceeded total items.
  Added test case to prevent regression.
  ```

Commit 3: Add email validation utility (1 file) ✅
  📝 src/utils/validation.ts
  Type: feat
  Scope: utils
  Priority: 1 (independent)
  Readiness: ✅ New utility function

  Suggested commit message:
  ```
  feat(utils): add email validation function
  ```

---

⚠️ Recommendations:
- Split these changes - they address different concerns
- Each commit is independent and complete
```

## Example 3: Work in Progress

### Input
```
Modified: src/payment/stripe.ts (has TODOs and console.logs)
New: src/payment/types.ts
Modified: src/payment/webhook.ts (incomplete error handling)
```

### Diff Analysis Shows
```typescript
// src/payment/stripe.ts
export async function processPayment(amount: number) {
  console.log('Processing payment:', amount); // TODO: remove debug
  // TODO: Add retry logic
  // TODO: Handle webhook failures

  const result = await stripe.charges.create({...});
  return result;
}

// src/payment/webhook.ts
export function handleWebhook(event: any) {
  // FIXME: Add proper error handling
  if (event.type === 'payment.success') {
    // Implementation incomplete
  }
}
```

### Output
```
📊 Git Changes Analysis (3 files)

---

🎯 Recommended Commit Strategy: 1 commit (NOT READY)

Commit 1: Start payment integration (3 files) 🚧
  📝 src/payment/stripe.ts
  🟢 src/payment/types.ts
  📝 src/payment/webhook.ts
  Type: feat
  Scope: payment
  Priority: 1 (independent)
  Readiness: 🚧 Work in Progress - NOT ready to commit

  Issues detected:
  - src/payment/stripe.ts: Contains TODO comments and debug console.log
  - src/payment/webhook.ts: Has FIXME comment and incomplete implementation
  - Missing error handling and retry logic

---

⚠️ Recommendations:
- Complete implementation before committing
- Remove debug logging
- Implement error handling as noted in TODOs
- Consider adding tests before committing
- Alternative: Commit to feature branch with "WIP:" prefix if needed

📋 Suggested Next Steps:
1. Address TODO and FIXME comments
2. Remove console.log statements
3. Add error handling
4. Write tests for payment flow
5. Then run /git_prepare_commits again
```

## Example 4: Dependency Chain

### Input
```
New: packages/utils/src/logger.ts
Modified: packages/api/src/server.ts (uses new logger)
Modified: packages/web/src/app.tsx (uses new logger)
Modified: packages/utils/package.json
```

### Output
```
📊 Git Changes Analysis (4 files)

---

🎯 Recommended Commit Strategy: 2 commits (ORDERED)

Commit 1: Add shared logger utility (2 files) ✅
  🟢 packages/utils/src/logger.ts
  📝 packages/utils/package.json
  Type: feat
  Scope: utils
  Priority: 1 (MUST BE FIRST - others depend on this)
  Readiness: ✅ Complete utility implementation

  Suggested commit message:
  ```
  feat(utils): add shared logger utility

  Provides consistent logging interface across packages.
  ```

Commit 2: Integrate logger in api and web packages (2 files) ✅
  📝 packages/api/src/server.ts
  📝 packages/web/src/app.tsx
  Type: feat
  Scope: api,web
  Priority: 2 (depends on commit 1)
  Readiness: ✅ Integration complete

  Suggested commit message:
  ```
  feat(api,web): integrate shared logger utility

  Replaces console.log with centralized logger.
  ```

---

⚠️ Recommendations:
- MUST commit in order: Commit 1 before Commit 2
- Commit 1 introduces dependency that Commit 2 uses
- Each commit should build and pass tests independently
```

## Example 5: Accidental Files

### Input
```
Modified: src/app.ts
New: nul
Modified: .gitignore
New: debug.log
```

### Output
```
📊 Git Changes Analysis (4 files)

---

🎯 Recommended Commit Strategy: 2 items (1 commit + 1 cleanup)

Action 1: Clean up accidental files 🗑️
  🗑️ nul (appears to be Windows null device redirect)
  🗑️ debug.log (temporary debug file)

  Recommended command:
  ```
  git clean -f nul debug.log
  ```

Commit 1: Update app and gitignore (2 files) ✅
  📝 src/app.ts
  📝 .gitignore
  Type: chore
  Scope: core
  Priority: 1 (after cleanup)
  Readiness: ✅ Ready after removing accidental files

  Suggested commit message:
  ```
  chore: update app configuration and gitignore
  ```

---

⚠️ Recommendations:
- Remove accidental files BEFORE committing
- File 'nul' suggests a Windows command error (should be 'nul 2>&1' or similar)
- Consider adding debug.log to .gitignore if not already present

📋 Next Steps:
1. Run: git clean -f nul debug.log
2. Verify src/app.ts changes are intentional
3. Then commit remaining changes
```

## Example 6: Claude Code Tooling Changes

### Input
```
Modified: .claude/commands/commit.md
Modified: .claude/skills/git-changes-analyzer/SKILL.md
New: .claude/agents/linter-agent.md
```

### Output
```
📊 Git Changes Analysis (3 files)

---

🎯 Recommended Commit Strategy: 1 commit

Commit 1: Update commit workflow and add linter agent (3 files) ✅
  📝 .claude/commands/commit.md
  📝 .claude/skills/git-changes-analyzer/SKILL.md
  🟢 .claude/agents/linter-agent.md
  Type: claude
  Scope: commit
  Priority: 1 (independent)
  Readiness: ✅ Complete - Claude Code tooling improvements

  Suggested commit message:
  ```
  claude(commit): improve commit workflow with linting support

  - Enhanced commit command to run linter before analysis
  - Added git-changes-analyzer skill documentation
  - Created linter-agent for automated code quality checks
  ```

---

⚠️ Recommendations:
- These are Claude Code infrastructure changes
- Use 'claude' prefix for commands, agents, skills, and settings
- Use 'docs' prefix only for user-facing documentation (README, guides)
```

## Example 7: Quick Summary Mode (for /startup_check)

### Input
Same as Example 2 (mixed concerns)

### Output (Quick Mode)
```
7 changes total (🟢 2 added, 📝 5 modified)

Topics: authentication refactor, API bug fixes, utility additions

Tell me if I should list files (list, yes)
```

### After User Says "list"
```
Changes by topic:

🔐 Authentication (1 file)
  - 📝 src/auth/login.ts

🔧 API fixes (2 files)
  - 📝 src/api/users.ts
  - 📝 tests/api/users.test.ts

🛠️ Utilities (1 file)
  - 📝 src/utils/validation.ts

💡 Tip: Run /git_prepare_commits for detailed commit strategy
```
