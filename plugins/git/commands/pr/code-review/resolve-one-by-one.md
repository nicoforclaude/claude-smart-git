---
description: "Read PR review comments, triage by severity, fix one at a time"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell), Skill(nico-dev:scope), Skill(nico-dev:coding), Skill(nico-dev:svelte5-developer), Skill(nico-dev:typescript-developer), Skill(linting:linting_fix)
---

# PR: Code Review — Resolve One by One

Read the latest code review on the current PR, analyze all comments, share LFH opinion, then work through issues one at a time.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Confirm we're on a PR branch

```bash
gh pr view --json number,headRefName,state,title
```

If no PR is found — stop and tell the user.

## Step 3 — Fetch all review comments

```bash
PR=$(gh pr view --json number --jq '.number')
REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner')
```

Fetch review-level comments (top-level review bodies):

```bash
gh api repos/$REPO/pulls/$PR/reviews --jq '[.[] | select(.body != "") | {id, state, body, user: .user.login, submitted_at}]'
```

Fetch inline comments (line-level):

```bash
gh api repos/$REPO/pulls/$PR/comments --jq '[.[] | {id, path, line, body, user: .user.login}]'
```

## Step 4 — Build the issue list

Combine both sources into a unified list. Deduplicate follow-up threads (keep the root comment, note reply count).

Categorize each issue:

- **Critical** — bugs, correctness issues, security, breaking contracts
- **LFH** (Low Hanging Fruit) — naming, small refactors, obvious omissions, trivial style
- **Normal** — design suggestions, non-trivial refactors, architectural feedback

## Step 5 — Share LFH opinion

Before starting any fixes, tell the user:

- Total issue count (breakdown by category)
- Which items are LFH and why (one line each)
- Recommended order: Critical → LFH → Normal

Use `AskUserQuestion`: "Ready to start? We'll go one at a time."

Wait for confirmation.

## Step 6 — Loop: one issue at a time

For each issue (Critical first, then LFH, then Normal):

### 6a — Present the issue

Show:

- Issue number (e.g., "Issue 2 of 7")
- Category
- The comment text and file/line if inline
- Your interpretation of what needs to change

### 6b — Fix

Activate `nico-dev:scope` and `nico-dev:coding` skills.
For Svelte files also activate `nico-dev:svelte5-developer`.
For TypeScript files also activate `nico-dev:typescript-developer`.

Read the relevant file(s) before editing. Make the change.

After each fix, run lint using `linting:linting_fix` skill.

### 6c — Confirm and continue

Show a brief summary of what was changed.

Use `AskUserQuestion`: "Done. Move to the next issue?"

- If yes — continue the loop.
- If no / stop / pause — exit cleanly, summarize progress (N of M issues resolved).

## Step 7 — Final summary

After the loop ends (all done or user stopped), show:

- Total issues resolved vs remaining
- List of files changed
- Reminder: run `/git:commit` when ready
