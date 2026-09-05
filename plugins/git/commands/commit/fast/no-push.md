---
description: "Quick commit without push — no linting, no change analysis, no confirmation prompts"
argument-hint: "[commit message]"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: Fast Commit, No Push

Same as fast commit but never pushes and never asks — commits straight away. Skips linter and changes-analyzer. Use when you know your changes and want maximum speed, keeping the commit local.

Still runs critical safety guards: admin test files, nul artifacts, main branch convention check.

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

## Step 5 — Main branch guard

**If current branch is `main`**, check for branch convention docs:
```bash
git ls-files | grep -i "branch"
```
Look for files matching `branch*.md` or `*branch-naming*`.

- **Convention doc found → warn, ask once:**
  Show: `⚠️ On \`main\` with branch convention in place. Commit stays local, but consider a feature branch.`
  Options: "Commit anyway", "Cancel"

- **No convention doc → proceed** (commit is local, low risk).

## Step 6 — Execute

1. Stage all changed files from `git status --short` (tracked modifications + meaningful untracked)
2. Commit:
```bash
git commit -m "$(cat <<'EOF'
[message]
EOF
)"
```
3. Report: `Committed: [hash] (not pushed)`
