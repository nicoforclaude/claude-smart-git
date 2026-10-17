---
description: "Auto PR pipeline — diagnoses the situation (branch / commit / push / PR) and executes it all after one Go/No Go dialog"
argument-hint: "[optional: topic hint for branch/PR naming]"
allowed-tools: Bash, Read, Edit, Write, AskUserQuestion, Skill(windows-shell:windows-shell), Skill(git:pr:watch), Skill(code-review)
---

# Git: PR Create Auto

Adaptive PR pipeline: diagnose → classify → one Go/No Go dialog → execute without further questions.

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
gh pr view --json url,title,number 2>/dev/null
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

## Step 3 — Diagnose

First infer the PR topic from `$ARGUMENTS` if provided, else from session context (files edited, subjects discussed).

Then answer five questions from Step 2 output:

| # | Question | How |
|---|----------|-----|
| D1 | Are we on a feature branch? | See feature branch check below |
| D2 | How many pending changes? | `git status --short` — count + file list |
| D3 | Commits on this branch not merged to main? | `git log origin/main..HEAD` — count + subjects |
| D4 | PR already open for this branch? | `gh pr view` |
| D5 | Are the D3 commits related to the PR topic? | Compare commit subjects (and diffs of recent big changes) against the inferred topic; estimate a relatedness % |

**Feature branch check (D1):**
1. Strip any user prefix from the branch name (`nico/working-2` → `working-2`, `feat/game-summary` → `game-summary`)
2. Extract the key slug words from the inferred topic (e.g. "game summary design" → `game`, `summary`, `design`)
3. If none of those words appear in the stripped branch name → not a feature branch for this work
4. Generic names (`working`, `working-N`, `scratch`, `sandbox`, `dev`, `temp`) are never feature branches, regardless of topic

### Situation classes

| Class | Condition | Go executes |
|-------|-----------|-------------|
| A | Not on feature branch (D1 = no) | branch → commit → push → PR |
| B | Feature branch, pending changes, no PR | commit → push → PR |
| C | Feature branch, clean, unpushed commits, no PR | push → PR |
| D | Feature branch, pushed, no PR | PR only |
| E | PR exists, pending or unpushed work | commit and/or push to existing PR |
| F | PR exists, nothing pending | no dialog — show PR URL, stop |
| G | Nothing pending, nothing ahead of main | no dialog — "Nothing to PR.", stop |

## Step 4 — Pre-commit safety checks (if commit needed)

If commit is in the pipeline, check staged/unstaged files:

| Check | Pattern | Action |
|-------|---------|--------|
| Admin Test Safety | `*.admin.test.ts` without `.skip()` or with `dryRun: false` | BLOCK until fixed |
| Windows nul Artifact | File named `nul` | BLOCK, offer to delete |

If blocked: explain which file and why. Wait for user to fix before continuing.

## Step 5 — Prepare values

**Branch name options** (class A only):
- Generate exactly 3 names following the convention found in Step 2 (prefix/user pattern from existing branches):
  - One **conservative** — obvious slug, standard type
  - One **descriptive** — captures the "why" not just the "what"
  - One **short** — tightest possible slug
- Kebab-case, lowercase, no filler words (`update`, `change`, `misc`)

**Commit message** (if commit needed):
- Infer conventional commit type from diff stat: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `style`
- One line, no body
- Strip AI attribution patterns (`/claude/i`, `/\bAI\b/`, `/generated with/i`, `/co-authored-by.*anthropic/i`, `/🤖/`)

**PR title and body** (if PR needed):
- Read any planning docs found in Step 2 that match branch/session topic
- Title: synthesize from session context and commit subjects, under 70 characters
- Body:
  ```
  ## Summary
  {bullets from session context and planning docs}

  ## Commits
  {commit subjects from git log origin/main..HEAD}
  ```

## Step 6 — One dialog: situation report + Go List

Classes F and G skip the dialog (see Step 3 table). For A–E, ask a single `AskUserQuestion` call with two questions: Go/No Go and Fixing mode.

**Do NOT print the report or plan as chat text before the tool call** — mid-turn text may not be displayed. Everything the user must see goes inside the call: report in the question text, plans in option previews.

**Question text** — the situation report (include only applicable lines):

```
Branch:   nico/working-2 — not a feature branch (generic name)
Pending:  7 files changed
Ahead:    2 commits not on main
Related:  ~90% related to "game summary" — fine to carry into the new branch
PR:       none
→ Recommend: new branch, carry commits along.
```

- `Related` appears only for class A with D3 > 0. If relatedness is low, say so and recommend discussing instead: `~20% related — they would pollute the PR; consider No Go to untangle first`
- End with a one-line `→ Recommend:` verdict

**Header:** "Go / No Go"

**Options by class** — every "Go" option is fully executable as-is:

| Class | Options |
|-------|---------|
| A | "Go: {name-1} (Recommended)" / "Go: {name-2}" / "Go: {name-3}" / "No Go — let's discuss" — each name option's description states its flavor (conservative / descriptive / short) |
| B, C, D | "Go (Recommended)" / "No Go — let's discuss" |
| E | "Go: update PR #{N} (Recommended)" / "No Go — let's discuss" |

**Preview** on each "Go" option — the plan with that option's values concrete (include only applicable steps):

```
🚀 Auto PR Pipeline

1. Branch  → feat/game-summary
2. Commit  → "feat(game): add game summary panel"
3. Push    → origin/feat/game-summary
4. PR      → "feat: game summary panel"

Files to stage:
  • path/to/file.ts
  • ...
```

If "No Go": stop executing and ask what they'd like to change — this is an invitation to discuss, not a silent exit.

**Question 2 — Fixing mode** (header: "Fixing mode"):

```
After the PR is up, how far should I take it?
```

| Option | Behavior |
|--------|----------|
| "Ready to merge (Recommended)" | Autonomous loop in Step 9: fix review findings and red CI, verify, push — until ready to merge or blocked on a decision |
| "As is — no fixes" | Watch CI + review once, report findings, stop |

**Choosing "Ready to merge" is explicit authorization to commit and push autonomously for this PR's fix loop** — no further confirmation per round.

## Step 7 — Execute

Run each planned step in sequence, reporting progress after each.

### Branch creation (class A)

The name was already chosen in Step 6 — no further interaction:

```bash
git checkout -b {chosen-name}
```

Uncommitted changes carry over to the new branch; commits ahead of main carry over too (the user accepted this via the `Related` line).

Report: `✅ Created branch: {chosen-name}`

### Commit (if planned)

Stage all tracked modifications and meaningful untracked files from `git status --short`, then:

```bash
git commit -m "$(cat <<'EOF'
{commit-message}
EOF
)"
```

Report: `✅ Committed: [{hash}] {message}`

### Inner review (before push)

One local review per PR — the cycle breaker: review depth depends on whether a review has already seen this branch.

- **Classes A–D (no PR yet):** invoke `Skill(code-review)` at **medium** effort on `origin/main..HEAD`.
  - Critical findings → stop, present them, wait for the user.
  - Warnings → report in one table and proceed.
  - Clean → proceed.
- **Class E (PR exists):** invoke `Skill(code-review)` at **low** effort on the unpushed diff only (`@{u}..HEAD`). **Report-only** — findings are shown but never block the push; the PR's CI review owns gating from here.

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

### PR creation (classes A–D)

```bash
gh pr create --title "{title}" --body-file - --base main <<'BODY'
{body}
BODY
```

Report: `✅ PR opened: {url}`

### Existing PR update (class E)

After commit/push, no PR creation needed.

Report: `✅ PR #{N} updated: {url}`

## Step 8 — Live monitor (CI + review)

After reporting the PR URL, invoke `Skill(git:pr:watch)` with the PR number. It polls CI checks and the automated review comment, fail-fast on red checks, and returns a triaged findings list (see its "Structured result" section).

**Mode "As is":** report the watch outcome (CI status + findings table with triage verdicts), then stop. Suggest `/git:pr:code-review:resolve-one-by-one` if items need reaction.

**Mode "Ready to merge":** continue to Step 9.

## Step 9 — Ready-to-merge loop (max 3 rounds)

Each round consumes the watch result:

| Watch result | Action |
|--------------|--------|
| CI green + every finding triaged `no action` | **Done — ready to merge.** Report the URL plus each dismissed finding with its one-line why (the user audits the subjective call) |
| Any `decision` finding | Stop the loop, present the tradeoff options, wait for the user |
| Review-job auth failure | Stop — follow pr:watch's remediation path (never auto-run secret updates) |
| `needs reaction` findings and/or red CI (tests/build/lint) | Fix round below |

**Ready to merge is a triage verdict, not a badge color:** a red/orange review status does not block when every remaining finding is judged `no action`. Red CI checks (tests, build, lint) always do.

### Fix round

1. **Fix** only the `needs reaction` items and CI breakage. Never re-attempt a finding the review re-flagged after a previous fix — that escalates to the user instead.
2. **Verify** — derive checks from CI, not guesses:
   - Read `.github/workflows/*.yml`; collect the run steps of jobs triggered on `pull_request`/`push` (test, check, lint, build).
   - Run those commands locally; all must pass before pushing.
   - Run `Skill(code-review)` at **low** effort on the fix diff only (`@{u}..HEAD`). Report-only — its findings never trigger another fix round.
3. **Commit and push** (authorized by the mode choice in Step 6), then invoke `Skill(git:pr:watch)` again.

After 3 rounds without reaching ready-to-merge: stop and summarize what is still red and why.
