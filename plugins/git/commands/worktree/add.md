---
description: "Add a new worktree as a numbered sibling slot (or named if topic given)"
argument-hint: "[optional: topic or existing branch name]"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: Worktree — Add

Create a new worktree as a sibling directory next to the current repo clone.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Read current state (parallel)

```bash
git rev-parse --show-toplevel
```

```bash
git worktree list
```

```bash
git branch -a --sort=-committerdate | head -20
```

```bash
git fetch origin main --quiet
```

## Step 3 — Derive repo identity

From `--show-toplevel` output:
- `REPO_ROOT` = full path (e.g., `C:\KolyaRepositories\nicoforclaude\claude-smart-git`)
- `REPO_NAME` = last path component (e.g., `claude-smart-git`)
- `PARENT_DIR` = parent of `REPO_ROOT` (e.g., `C:\KolyaRepositories\nicoforclaude`)

## Step 4 — Determine worktree directory name

**If `$ARGUMENTS` provided:**
- Slugify: lowercase, replace spaces and `/` with `-`, strip owner prefix (e.g. `nico/feature/auth` → `auth`)
- `WORKTREE_DIR = "{REPO_NAME}-{slug}"` (e.g. `claude-smart-git-auth`)

**If no argument — find next free numbered slot:**

List siblings matching `{REPO_NAME}-{N}` (N is a number):

```bash
powershell -Command "Get-ChildItem -Path '{PARENT_DIR}' -Directory | Where-Object { $_.Name -match '^{REPO_NAME}-\d+$' } | Select-Object -ExpandProperty Name"
```

Take `max(N) + 1`, defaulting to `2` if none exist.

- `WORKTREE_DIR = "{REPO_NAME}-{N}"` (e.g. `claude-smart-git-2`)

`WORKTREE_PATH = "{PARENT_DIR}/{WORKTREE_DIR}"`

## Step 5 — Determine branch

**If `$ARGUMENTS` matches an existing branch** — use it as-is (checkout existing branch, skip base).

**If `$ARGUMENTS` is a topic slug (not an existing branch):**
- Infer naming convention from existing branches (prefix pattern, e.g. `nico/feature/`)
- Apply convention to the slug (e.g. `nico/feature/auth`)

**If no argument:**
- Infer topic from session context (files edited, subjects discussed)
- If session context found → apply naming convention to inferred topic
- If no context → use `slot-{N}` as branch name (e.g. `slot-2`)

## Step 6 — Confirm with user

Use `AskUserQuestion` to present the plan:

```
📁 Worktree:  ../claude-smart-git-2    (sibling directory — lives next to your main clone)
🌿 Branch:    slot-2                   (new, based on origin/main)
```

Options:
- **Create** — proceed as shown
- **Change directory name** — ask for a custom name
- **Change branch name** — ask for a custom branch name

If user wants to change either, update the plan and show it again before proceeding.

## Step 7 — Create worktree

**New branch:**
```bash
git worktree add -b "{branch}" "{WORKTREE_PATH}" origin/main
```

**Existing branch:**
```bash
git worktree add "{WORKTREE_PATH}" "{branch}"
```

## Step 8 — Report

```
✅ Worktree created
   Path:    ../claude-smart-git-2
   Branch:  slot-2
```
