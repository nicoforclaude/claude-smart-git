---
description: "Create a PR — infers title and body from session context, commits, and planning docs"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# PR: Create

Create a pull request for the current branch. Infers title and body from session context, commits, and planning docs. Does not push if already up to date.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Check branch state (parallel)

```bash
git branch --show-current
git fetch origin --quiet
git log origin/main..HEAD --oneline
git status --short
```

```bash
# Look for planning docs
find . -maxdepth 6 \( -path "*/planning/*.md" -o -path "*/plans/*.md" \) 2>/dev/null | grep -v ".git/"
```

If on `main` — stop: "Cannot create a PR from `main`. Switch to a feature branch first."

If no commits ahead of main — stop: "No commits ahead of main. Nothing to PR."

If there are uncommitted changes — warn: "⚠️ You have uncommitted changes — they will not be included in the PR."

If planning docs found — read any that appear relevant to this branch's work (match by topic/name against branch name and commit subjects).

## Step 3 — Push if needed

Check if the branch has an upstream:

```bash
git rev-parse --abbrev-ref @{u} 2>/dev/null
```

If no upstream — push and set it:

```bash
git push --set-upstream origin {branch}
```

If upstream exists but is behind local — push:

```bash
git push
```

## Step 4 — Infer title and body

**Title**: Synthesize from:
1. **Session context** — what was worked on in this conversation (files edited, topics discussed, purpose of the work)
2. **Commits ahead of main** — commit subjects from Step 2

Prefer the session framing if it's richer. Fall back to the first commit subject if session context is thin. Keep under 70 characters.

**Body**: Build from all three sources:

```
## Summary
- {bullet per meaningful change — from session context and planning docs}

## Commits
- {each commit subject from git log}
```

Planning docs contribute to the Summary bullets — pull out the intent, goals, or key decisions described there. Keep bullets concise. No AI attribution. No filler.

## Step 5 — Confirm title

Show the inferred title and ask the user to confirm or replace it:

Use `AskUserQuestion`:
- Question: "PR title — confirm or type your own:"
- Option: "{inferred title}"
- Option: "Cancel"

The user can select the inferred title or type a custom one via the free-text field.

## Step 6 — Show preview and confirm

Show the full preview with the confirmed title:

```
📋 PR Preview:

Branch:  {branch} → main
Title:   {title}

Body:
{body}
```

Use `AskUserQuestion`:
- "Create PR"
- "Cancel"

## Step 7 — Create

Write the body to a temp file first to safely handle multi-line markdown and any special characters:

```bash
gh pr create --title "{title}" --body-file - --base main <<'BODY'
{body}
BODY
```

Report the PR URL.
