---
description: "Quick single commit — no linting, no change analysis, straight to commit"
argument-hint: "[commit message]"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: Fast Commit

Minimal commit path — skips linter and changes-analyzer agent. Use when you know your changes well and want speed.

Still runs critical safety guards: admin test files, nul artifacts.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Read current state

```bash
git branch --show-current
git status --short
git diff --stat
```

If working tree is clean → report "Nothing to commit." and stop.

## Step 3 — Pre-Commit Safety Checks

Check files in git status:

| Check | Pattern | Action |
|-------|---------|--------|
| Admin Test Safety | `*.admin.test.ts` without `.skip()` or with `dryRun: false` | BLOCK until fixed |
| Windows nul Artifact | File named `nul` | BLOCK, offer to delete |

If blocked: explain which file and why. Wait for user to fix before continuing.

## Step 4 — Build commit message

**If `$ARGUMENTS` provided** — use as the commit message verbatim.

**If no arguments** — infer a conventional commit message from the diff stat output:
- Pick the right type: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `style`
- One line, no body

**Strip AI attribution** from the message before proceeding:

Scan for these patterns and remove any matching lines or phrases:
- `/claude/i`
- `/\bAI\b/`
- `/generated with/i`
- `/co-authored-by.*anthropic/i`
- `/🤖/`

If anything was stripped, note: `ℹ️ Removed AI attribution from commit message`

## Step 5 — Show preview and ask

Always show target branch prominently:

```
📋 Fast Commit:

Branch: {current-branch}

Files to stage:
  • [list from git status --short]

Commit message:
  "[message]"
```

**If current branch is `main`**, check for branch convention docs first:
```bash
git ls-files | grep -i "branch"
```
Look for files matching `branch*.md` or `*branch-naming*`.

- **Convention doc found → hard guard:**
  Show: `⚠️ On \`main\` with branch convention in place. Push blocked — use a feature branch and PR.`
  Options: "Commit only", "Edit message", "Cancel"

- **No convention doc → soft warning:**
  Show: `⚠️ You are on \`main\`. Consider using a feature branch.`
  Options (push demoted to last): "Commit only", "Edit message", "Commit and push", "Cancel"

**Otherwise**, use AskUserQuestion:
- "Commit and push"  ← default/first
- "Commit only"
- "Edit message"
- "Cancel"

## Step 6 — Execute

**If "Edit message"**: Ask user to provide the new commit message, then re-show preview (Step 5) and ask again.

**If "Commit only" or "Commit and push"**:

1. Stage all changed files from `git status --short` (tracked modifications + meaningful untracked)
2. Commit:
```bash
git commit -m "$(cat <<'EOF'
[message]
EOF
)"
```
3. Report: `Committed: [hash]`
4. If push selected: run `git push` → report `Pushed to origin/[branch]` or show error as-is

**If "Cancel"**: Report "Cancelled. Nothing committed." and stop.
