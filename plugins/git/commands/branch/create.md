---
description: "Smart branch creation — finds naming convention, infers topic from session, creates branch"
argument-hint: "[optional: topic description]"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: Branch Create

Infer a well-named branch from session context or user input, check base health, and create it.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Read current state (parallel)

```bash
# Branch position vs main
git branch --show-current
git fetch origin main --quiet
git log HEAD..origin/main --oneline
git log origin/main..HEAD --oneline
```

```bash
# Pending changes
git status --short
git diff --stat HEAD
```

```bash
# Existing branches for convention inference
git branch -a --sort=-committerdate | head -20
```

## Step 3 — Find naming convention

Search the project for a branch naming convention doc:

```bash
find . -maxdepth 4 -name "branch*.md" -o -name "*branch-naming*" -o -name "*conventions*" 2>/dev/null | grep -v ".git/"
```

If found — read it and extract the naming pattern.

If not found — infer convention from existing branch names (format, prefix, slug style).

## Step 4 — Orient the user

Report current position:

```
Current branch: {branch-name}
Ahead of main:  N commits
Behind main:    N commits
Pending changes: {summary}
```

Assess base health:

| Situation | Assessment |
|---|---|
| On `main` or 0 commits ahead | ✅ Clean base |
| Feature branch, 0 commits ahead of main | ✅ Fine |
| Feature branch, ahead of main | ⚠️ New branch inherits unrelated commits |
| Behind main significantly | ⚠️ Consider merging main first |

If ⚠️ — explain the risk and ask: "Still branch from here, or switch to main first?" Wait for answer.

## Step 5 — Infer topic

**If `$ARGUMENTS` provided** — use that as the topic description, skip to Step 6.

**If session context is available** — summarize what was worked on:
- Files edited/created
- Subjects discussed
- Purpose of the work

Present the inferred topic: "Based on this session, you've been working on: [topic]. Does that sound right?"

**If unclear or no session context** — ask the user: "What are you working on? (one short phrase)"

## Step 6 — Suggest branch names

Generate exactly **3 options** following the inferred convention:

- One **conservative** — obvious slug, standard type
- One **descriptive** — captures the "why" not just the "what"
- One **short** — tightest possible slug

Rules:
- Kebab-case, lowercase
- No filler words (`update`, `change`, `misc`, `fix-stuff`)
- Reflects purpose, not implementation detail
- Follow the prefix/user pattern from existing branches

Use `AskUserQuestion` to let the user pick (or type their own).

## Step 7 — Create the branch

```bash
git checkout -b {chosen-branch-name}
```

Confirm the branch was created and working tree state carried over cleanly.
