# Common Commit Splitting Patterns

This document catalogs common patterns for how to split (or not split) changes into commits.

## Pattern: Feature with Tests and Docs

**Keep Together** ✅

```
feat(scope): add feature X
  - Implementation
  - Tests
  - Documentation
```

**Rationale**: These form a complete, reviewable unit. Splitting would create incomplete commits.

## Pattern: Claude Code Tooling Changes

**Use 'claude' prefix** ✅

```
# For commands, agents, skills, settings:
claude(commit): improve commit workflow with linting
claude(skills): add git-changes-analyzer skill
claude(agents): create linter-agent for code quality

# For user-facing documentation:
docs: update README with installation instructions
docs(guide): add troubleshooting section
```

**Rationale**: Distinguish Claude Code infrastructure (`claude`) from user documentation (`docs`). This makes it clear when changes affect the tooling itself vs. user-facing content.

## Pattern: Multiple Independent Features

**Split Apart** ❌→✅

```
# Instead of one commit:
feat: add feature X and feature Y

# Split into:
feat(x): add feature X
feat(y): add feature Y
```

**Rationale**: Unrelated features should be separate commits for easier review and potential revert.

## Pattern: Refactor + Feature

**Split Apart** ❌→✅

```
# Instead of:
feat: add feature X with refactored code

# Split into:
refactor(module): restructure for extensibility
feat(module): add feature X
```

**Rationale**: Refactors should be reviewable independently. Feature depends on refactor, so commit refactor first.

## Pattern: Fix + Related Test

**Keep Together** ✅

```
fix(api): correct validation logic
  - Fix in source
  - Regression test
```

**Rationale**: Fix and its test are tightly coupled. Together they demonstrate the bug and its resolution.

## Pattern: Dependency Update + Breaking Changes

**Split Apart** ❌→✅

```
# Split into:
build(deps): update library X to v2.0
refactor: adapt to library X v2.0 breaking changes
```

**Rationale**: Dependency update is mechanical; adaptations show business impact. Easier to review separately.

## Pattern: Multiple Small Fixes in Same File

**Consider File Purpose** ⚖️

If same concern:
```
fix(auth): correct multiple validation edge cases
```

If different concerns:
```
fix(auth): correct email validation
fix(auth): handle expired tokens properly
```

**Rationale**: Group by logical purpose, not by file location.

## Pattern: Rename/Move + Modify

**Split Apart** ❌→✅

```
# First commit (pure rename/move):
refactor: rename UserService to AccountService

# Second commit (modifications):
feat(account): add new account features
```

**Rationale**: Renames/moves should be pure (no content changes) for easier diff review.

## Pattern: Config Changes + Implementation

**Consider Dependency** ⚖️

If config is prerequisite:
```
Commit 1: chore(config): add environment variables for feature X
Commit 2: feat(x): implement feature X
```

If config is part of feature:
```
feat(x): implement feature X
  - Implementation
  - Configuration
```

**Rationale**: Depends on whether config has meaning independent of feature.

## Pattern: Multiple Package Changes (Monorepo)

**Group by Logical Change** ✅

```
feat(utils,api,web): add shared authentication

Changes:
  - packages/utils/src/auth.ts (new utility)
  - packages/api/src/middleware.ts (use utility)
  - packages/web/src/auth.tsx (use utility)
```

**Rationale**: Cross-package features that implement one logical change should stay together.

## Pattern: WIP + Complete Work

**Split Apart** ❌→✅

```
# Commit complete work:
feat(analytics): add event tracking

# Hold WIP for later:
[Not committed yet - WIP]
  - src/analytics/advanced-reporting.ts (incomplete)
```

**Rationale**: Never mix WIP with complete work. Commit complete work; continue WIP in working tree.

## Pattern: Style/Format + Logic Changes

**Split Apart** ❌→✅

```
# First commit (pure style):
style(core): apply prettier formatting

# Second commit (logic):
feat(core): add new validation rules
```

**Rationale**: Style changes create noise in diffs. Separate them for clearer review of logic changes.

## Pattern: Debug Code + Real Changes

**Remove Debug, Then Commit** 🗑️→✅

```
# Remove:
  - console.log statements
  - commented code
  - temporary debug files

# Then commit clean changes:
feat(feature): add feature X
```

**Rationale**: Debug artifacts should never be committed. Clean up first.

## Pattern: Emergency Hotfix + Unrelated WIP

**Stash WIP, Commit Hotfix** 💾→✅

```
# Stash WIP:
git stash

# Commit only hotfix:
fix(critical): resolve production issue

# Restore WIP:
git stash pop
```

**Rationale**: Hotfixes must be clean and targeted. Don't pollute with unrelated WIP.

## Pattern: Generated Files + Source

**Depends on Commit Policy** ⚖️

If generated files are committed (e.g., package-lock.json):
```
feat(api): add endpoint X
  - src/api/endpoint.ts
  - package-lock.json (updated due to new dependency)
```

If generated files are gitignored:
```
feat(api): add endpoint X
  - src/api/endpoint.ts
  (package-lock.json not committed)
```

**Rationale**: Follow project policy. Most modern projects don't commit generated files.

## Pattern: Breaking Change + Migration

**Keep Together or Clear Sequence** ✅

Option 1 (together):
```
feat(api)!: redesign user endpoint

BREAKING CHANGE: User endpoint now requires authentication.
Updated all consumers to pass auth tokens.
```

Option 2 (sequence):
```
Commit 1: feat(api): add auth support to user endpoint (backward compatible)
Commit 2: feat(api)!: require auth for user endpoint
Commit 3: chore: remove deprecated non-auth code paths
```

**Rationale**: Breaking changes need clear migration path. Document in commit message or commits.

## Anti-Patterns to Avoid

### ❌ "Cleanup" Commits
```
# Bad:
chore: cleanup and fixes
  - Fixed 3 bugs
  - Refactored 2 modules
  - Updated docs
  - Removed debug code
```
**Why**: Impossible to review or understand. Split into logical commits.

### ❌ Mixing Scope Changes
```
# Bad:
feat: various improvements
  - auth/login.ts
  - payment/stripe.ts
  - ui/header.tsx
  (all unrelated changes)
```
**Why**: Changes in different scopes should be separate commits.

### ❌ Committing TODOs
```
# Bad:
feat: add payment processing
  // TODO: Add error handling
  // FIXME: This is broken
  console.log('debug this later')
```
**Why**: Incomplete work shouldn't be committed to main branches.

### ❌ "Friday Afternoon" Commits
```
# Bad:
wip: stuff before weekend
  (20 files changed, mix of everything)
```
**Why**: Even WIP should be organized. Use feature branches or split properly.

## Decision Tree

```
Is this one logical change?
├─ Yes: Keep together
│  └─ Does it include tests/docs?
│     ├─ Yes: Keep together ✅
│     └─ No: Consider adding them
│
└─ No: Consider splitting
   ├─ Are changes in same scope/module?
   │  ├─ Yes: Consider grouping
   │  └─ No: Split apart ✅
   │
   └─ Is there dependency between changes?
      ├─ Yes: Commit in order ✅
      └─ No: Independent commits ✅
```

## Readiness Checklist

Before committing, verify:
- [ ] No TODO/FIXME/WIP comments
- [ ] No debug logging (console.log, print, etc.)
- [ ] No commented-out code
- [ ] All related changes included
- [ ] Tests pass (if applicable)
- [ ] Follows project conventions
- [ ] Single, clear purpose
- [ ] Reviewable size (not too large)
- [ ] No accidental files (nul, debug.log, etc.)
- [ ] Commit message is clear and accurate
