---
description: "Delete all bot review comments and dismiss all human reviews on this PR"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# PR: Code Review — Cleanup (All)

Remove every bot code review comment and dismiss all human reviews. Use when resetting the review slate entirely.

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

**Bot issue comments**:
```bash
gh api --paginate "repos/$REPO/issues/$PR/comments" \
  --jq '[.[] | select(.user.type == "Bot") | {id, created_at, user: .user.login}]'
```

**Bot inline review comments**:
```bash
gh api --paginate "repos/$REPO/pulls/$PR/comments" \
  --jq '[.[] | select(.user.type == "Bot") | {id, created_at, user: .user.login}]'
```

**Human reviews**:
```bash
gh api --paginate "repos/$REPO/pulls/$PR/reviews" \
  --jq '[.[] | select(.user.type != "Bot" and .state != "PENDING") | {id, state, submitted_at, user: .user.login}]'
```

## Step 5 — Show summary and confirm

Display:
- Total bot issue comments found
- Total bot inline comments found
- Total human reviews found

If nothing found — tell the user and exit.

Use `AskUserQuestion`: "This deletes ALL bot comments and dismisses ALL human reviews. Proceed?"

If no — exit without changes.

## Step 6 — Execute

Delete each bot issue comment:
```bash
gh api -X DELETE "repos/$REPO/issues/comments/$COMMENT_ID"
```

Delete each bot inline comment:
```bash
gh api -X DELETE "repos/$REPO/pulls/comments/$COMMENT_ID"
```

Dismiss each human review:
```bash
gh api -X PUT "repos/$REPO/pulls/$PR/reviews/$REVIEW_ID/dismissals" \
  -f message="Cleanup: all reviews dismissed" -f event="DISMISS"
```

## Step 7 — Report

Show totals:
- Bot issue comments deleted
- Bot inline comments deleted
- Human reviews dismissed
