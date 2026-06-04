---
description: "Auto PR pipeline — detects what's needed (branch / commit / push / PR) and does it all in one shot"
argument-hint: "[optional: topic hint for branch/PR naming]"
allowed-tools: Bash, Read, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# Git: PR Create Auto

Adaptive PR pipeline. Detects which steps are still needed, previews the full plan in one shot, then executes everything with a single confirmation.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Inspect state (parallel)

```bash
git branch --show-current
git fetch origin --quiet
git log origin/main..HEAD --oneline
git status --short
git diff --stat HEAD
```

```bash
# Upstream state
git rev-parse --abbrev-ref @{u} 2>/dev/null
git log @{u}..HEAD --oneline 2>/dev/null
```

```bash
# Existing PR
gh pr view --json url,title 2>/dev/null
```

```bash
# Branch naming convention + recent branches
git branch -a --sort=-committerdate | head -20
find . -maxdepth 4 \( -name "branch*.md" -o -name "*branch-naming*" -o -name "*conventions*" \) 2>/dev/null | grep -v ".git/"
```

```bash
# Planning docs
find . -maxdepth 6 \( -path "*/planning/*.md" -o -path "*/plans/*.md" \) 2>/dev/null | grep -v ".git/"
```

## Step 3 — Determine required steps

| Condition | Required steps |
|-----------|----------------|
| On `main`, has uncommitted changes | branch → commit → push → PR |
| On feature branch, has uncommitted changes | commit → push → PR |
| On feature branch, committed but not pushed | push → PR |
| On feature branch, pushed, no PR | PR only |
| On feature branch, pushed, PR exists | STOP: show existing PR URL |
| Nothing to commit, no commits ahead of main | STOP: "Nothing to PR." |

## Step 4 — Pre-commit safety checks (if commit needed)

If commit is in the required steps, check staged/unstaged files:

| Check | Pattern | Action |
|-------|---------|--------|
| Admin Test Safety | `*.admin.test.ts` without `.skip()` or with `dryRun: false` | BLOCK until fixed |
| Windows nul Artifact | File named `nul` | BLOCK, offer to delete |

If blocked: explain which file and why. Wait for user to fix before continuing.

## Step 5 — Prepare values

**Branch name** (if branch creation needed):
- Read convention doc if found; otherwise infer from existing branch names
- Infer topic from `$ARGUMENTS` if provided, else from session context (files edited, work discussed)
- Generate **one** best-fit name: kebab-case, lowercase, follows convention, reflects purpose

**Commit message** (if commit needed):
- Infer conventional commit type from diff stat: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `style`
- One line, no body
- Strip AI attribution patterns (`/claude/i`, `/\bAI\b/`, `/generated with/i`, `/co-authored-by.*anthropic/i`, `/🤖/`)

**PR title and body** (always):
- Read any planning docs found in Step 2 that match branch/session topic
- Title: synthesize from session context and commit subjects, under 70 characters
- Body:
  ```
  ## Summary
  {bullets from session context and planning docs}

  ## Commits
  {commit subjects from git log origin/main..HEAD}
  ```

## Step 6 — Show plan and confirm

Present a single confirmation with all concrete values:

```
🚀 Auto PR Pipeline

Current branch: {branch}

Steps:
  [1. Create branch:  feat/your-feature-name]   ← only if on main
  [2. Commit:         "feat(scope): message"]    ← only if uncommitted changes
  [3. Push to origin]                            ← only if not pushed
  [4. Open PR:        "feat: your title"]

[Files to stage:                                 ← only if committing
  • path/to/file.ts
  • ...]
```

Use `AskUserQuestion`:
- "Proceed"
- "Cancel"

If "Cancel": stop.

## Step 7 — Execute

Run each planned step in sequence, reporting progress after each.

### Branch creation (if planned)

```bash
git checkout -b {branch-name}
```

Report: `✅ Created branch: {branch-name}`

### Commit (if planned)

Stage all tracked modifications and meaningful untracked files from `git status --short`, then:

```bash
git commit -m "$(cat <<'EOF'
{commit-message}
EOF
)"
```

Report: `✅ Committed: [{hash}] {message}`

### Push (if planned)

If no upstream:

```bash
git push --set-upstream origin {branch}
```

If upstream exists but behind local:

```bash
git push
```

Report: `✅ Pushed to origin/{branch}`

### PR creation

```bash
gh pr create --title "{title}" --body-file - --base main <<'BODY'
{body}
BODY
```

Report: `✅ PR opened: {url}`
