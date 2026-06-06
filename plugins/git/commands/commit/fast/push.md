---
description: "Quick commit and push — no linting, no change analysis, no confirmation prompts"
argument-hint: "[commit message]"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: Fast Commit and Push

Same as fast commit but always pushes — no confirmation prompt. Skips linter and changes-analyzer. Use when you know your changes and want maximum speed.

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

- **Convention doc found → hard block:**
  Show: `⚠️ On \`main\` with branch convention in place. Push blocked — use a feature branch and PR.`
  Stop. Do not commit or push.

- **No convention doc → warn and stop:**
  Show: `⚠️ On \`main\`. This command always pushes — aborting to be safe. Use /git:commit:fast if you want to commit only.`
  Stop.

## Step 6 — Execute

1. Stage all changed files from `git status --short` (tracked modifications + meaningful untracked)
2. Commit:
```bash
git commit -m "$(cat <<'EOF'
[message]
EOF
)"
```
3. Report: `Committed: [hash]`
4. Push:
```bash
git push
```
5. Report: `Pushed to origin/[branch]` or show error as-is.
