---
description: "Show unmerged local branches with recency, drift from main, and PR status"
allowed-tools: Bash, Skill(windows-shell:windows-shell)
---

# Git: Show Unmerged Branches

Read-only overview of local branches not merged into main: how fresh they are, how far they drifted, and whether a PR sits on them. No prompts, no changes.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Gather data (parallel)

Run these in parallel:

```bash
# Unmerged local branches (note * current and + worktree markers)
git fetch origin --prune --quiet
git branch --no-merged origin/main
```

```bash
# Recency per branch, newest first
git for-each-ref refs/heads --sort=-committerdate --format="%(refname:short)|%(committerdate:relative)"
```

```bash
# All PRs once (skip PR column if gh fails or no GitHub remote)
gh pr list --state all --limit 200 --json number,state,headRefName
```

Then for each unmerged branch, get drift:

```bash
git rev-list --left-right --count origin/main...{branch}
```

(first number = behind main, second = ahead)

## Step 3 — Report

One table, sorted newest commit first. Match PRs by `headRefName`; use `open #N` / `merged #N` / `closed #N`, or `—` if none.

```
| Branch | Last commit | Ahead / Behind | PR |
|---|---|---|---|
| nico/feat/layout-command | 2 days ago | 4 ↑ / 12 ↓ | open #218 |
| nico/wip/layout-modes ⧉ | 3 weeks ago | 7 ↑ / 90 ↓ | — |
```

Mark worktree-checked-out branches (`+` in git output) with `⧉`, the current branch with `*`.

After the table, add hints only if they apply:

- Branches with a **merged** PR: likely squash-merge leftovers — `git branch --merged` can't detect these; suggest deleting manually or with `git:branch:cleanup`.
- Branches with **0 ahead**: contain nothing main doesn't have — safe to delete.

If there are no unmerged branches: "All local branches are merged into main. ✅"
