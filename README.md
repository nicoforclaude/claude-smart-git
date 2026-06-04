# Claude Smart Git

> A Claude Code plugin that gives you a complete, intelligent git workflow — from smart commit analysis to PR management and worktree coordination.

Stop thinking about git mechanics. `/commit` analyzes your changes, groups them logically, lints, generates a meaningful message, and handles hooks. The rest of the 18 commands cover branches, PRs, worktrees, and daily sync — all driven by Claude.

---

## Installation

**Prerequisites:** Claude Code CLI with plugin support. Windows users also need the [windows-shell](https://github.com/nicoforclaude/claude-windows-shell) plugin.

```shell
# Add the marketplace
/plugin marketplace add https://github.com/nicoforclaude/claude-smart-git

# Install the plugin
/plugin install git@claude-smart-git
```

Verify: `/plugin list`

### Upgrading from v0.1.0

```shell
/plugin uninstall git
# then reinstall above
```

---

## Commands

### Commits

| Command | What it does |
|---------|-------------|
| `/git:commit` | Full smart commit — linting, change analysis, AI message generation, hook handling |
| `/git:commit:fast` | Quick commit — skips linting and analysis, straight to message and commit |

### Daily Flow

| Command | What it does |
|---------|-------------|
| `/git:startup` | Git status check at session start — branch, staged, unstaged, untracked |
| `/git:status` | Current branch and change overview |
| `/git:pull-main` | Fetch and merge `origin/main` into current branch |
| `/git:push` | Push commits to remote |
| `/git:sync` | Diagnose and sync branches downstream (deploy → main → working) |
| `/git:to-main` | Switch to main with safety check for pending changes |

### Branches

| Command | What it does |
|---------|-------------|
| `/git:branch:create` | Smart branch creation — infers naming convention and topic from session |
| `/git:branch:cleanup` | Delete current branch once merged — lands on main or working branch |
| `/git:branch:cleanup:scan` | Scan for merged branches, multi-select bulk cleanup |

### Pull Requests

| Command | What it does |
|---------|-------------|
| `/git:pr:create` | Create PR — infers title and body from session context and commits |
| `/git:pr:fix-ci` | Fetch failing CI logs, reproduce locally, fix and verify (no commit) |
| `/git:pr:code-review:resolve-one-by-one` | Read PR comments, triage by severity, fix one at a time |
| `/git:pr:code-review:cleanup:all` | Delete all bot review comments and dismiss all human reviews |
| `/git:pr:code-review:cleanup:keep-last` | Delete all bot comments except latest round, dismiss older reviews |

### Worktrees

| Command | What it does |
|---------|-------------|
| `/git:worktree:add` | Add a new worktree as a numbered sibling slot (or named if topic given) |
| `/git:worktree:show` | List all worktrees with branch and pending changes status |
| `/git:worktree:pull-main` | Fetch `origin/main` and fast-forward all clean worktrees |

---

## Smart Commit Analysis

The `changes-analyzer` skill (auto-triggered by `/git:commit`) evaluates every file before it touches `git commit`:

| Indicator | Meaning |
|-----------|---------|
| ✅ Ready | Changes are coherent and self-contained |
| 🚧 In Progress | Work is incomplete or mixed concerns |
| ⚠️ Needs Review | Suspicious patterns detected |
| 🗑️ Cleanup | Temporary or debug code present |
| 🚫 Do Not Commit | Secrets, breaking changes, or unsafe content |

When changes span multiple logical concerns, it recommends atomic commit groupings — separating features from fixes, refactoring from functionality — before you commit anything.

---

## Version History

- **0.2.0** — Migrated to plugin marketplace architecture (`.claude-plugin` structure). **Breaking change:** requires plugin installation method.
- **0.1.0** — Initial release: changes-analyzer skill, agent, and 5 commands.
