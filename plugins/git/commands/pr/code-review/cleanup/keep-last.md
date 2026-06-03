---
description: "Delete all bot review comments except the latest round, dismiss older human reviews"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# PR: Code Review — Cleanup (Keep Last)

Remove stale bot code review comments, keeping only the most recent round. Dismiss older human reviews. Useful after completing a review cycle and starting a new one.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Confirm we're on a PR branch

```bash
gh pr view --json number,headRefName,state,title
```

If no PR is found — stop and tell the user.

## Step 3 — Collect PR metadata

```bash
PR=$(gh pr view --json number --jq '.number')
REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner')
```

## Step 4 — Fetch comments and reviews

**Bot issue comments** (general PR comments from bots):
```bash
gh api --paginate "repos/$REPO/issues/$PR/comments" \
  --jq '[.[] | select(.user.type == "Bot") | {id, created_at, user: .user.login}]'
```

**Bot inline review comments**:
```bash
gh api --paginate "repos/$REPO/pulls/$PR/comments" \
  --jq '[.[] | select(.user.type == "Bot") | {id, pull_request_review_id, created_at, user: .user.login}]'
```

**Human reviews**:
```bash
gh api --paginate "repos/$REPO/pulls/$PR/reviews" \
  --jq '[.[] | select(.user.type != "Bot" and .state != "PENDING") | {id, state, submitted_at, user: .user.login}]'
```

## Step 5 — Identify what to keep

**Bot issue comments:** Sort by `created_at` descending. Keep the single newest. Mark all others for deletion.

**Bot inline comments:** Group by `pull_request_review_id`. Keep comments from the group with the highest review ID. Mark all others for deletion.

**Human reviews:** Sort by `submitted_at` descending. Keep the newest. Mark all others for dismissal.

If nothing to clean up — tell the user and exit.

## Step 6 — Show summary and confirm

Display:
- Bot issue comments: N to delete, 1 to keep (posted at `<time>`)
- Bot inline comments: N to delete, M to keep (from review `<id>`)
- Human reviews: N to dismiss, 1 to keep (by `<user>` at `<time>`)

Use `AskUserQuestion`: "Proceed with cleanup?"

If no — exit without changes.

## Step 7 — Execute

Delete each bot issue comment marked for removal:
```bash
gh api -X DELETE "repos/$REPO/issues/comments/$COMMENT_ID"
```

Delete each bot inline comment marked for removal:
```bash
gh api -X DELETE "repos/$REPO/pulls/comments/$COMMENT_ID"
```

Dismiss each human review marked for dismissal:
```bash
gh api -X PUT "repos/$REPO/pulls/$PR/reviews/$REVIEW_ID/dismissals" \
  -f message="Superseded by latest review" -f event="DISMISS"
```

## Step 8 — Report

Show:
- Bot issue comments deleted
- Bot inline comments deleted
- Human reviews dismissed
- What was kept
