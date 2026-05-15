---
description: "Fetch failing CI logs, reproduce locally, fix, and verify — does not commit"
argument-hint: "<keyword>  (e.g. lint, svelte, test)"
allowed-tools: Bash, AskUserQuestion, Skill(windows-shell:windows-shell), Skill(linting:linting_fix), Skill(nico-dev:svelte5-developer), Skill(nico-dev:scope), Skill(nico-dev:coding)
---

# PR: Fix CI

Resolve the keyword to a CI check, fetch the failing logs, reproduce locally, fix, and verify.
Does not commit.

## Step 1 — Load Windows Shell Skill (Windows Only)

If running on Windows, load `windows-shell:windows-shell` skill.

## Step 2 — Read context (parallel)

Run in parallel:

```bash
# Confirm we're on a PR branch
gh pr view --json number,headRefName,state
```

```bash
# Look for per-repo CI map
find . -maxdepth 3 -path "*/.claude/ci-map.md" 2>/dev/null
```

If no PR found — stop: "No open PR found for this branch."

If ci-map found — read it and build a lookup table of keyword → `{ check-name, fix, verify, type }`.

If ci-map not found — use the keyword directly as both check name and type, with no preset local commands.

## Step 3 — Resolve keyword

**If `$ARGUMENTS` provided** — look it up in the ci-map (match against `keywords:` list, case-insensitive).

If no match found in ci-map — treat the argument as the check name directly and warn:
> "ℹ️ No ci-map entry found for `{keyword}`. Using it as the check name directly. Local commands unknown."

**If no argument** — list all entries from ci-map and ask user to pick using `AskUserQuestion`.
If no ci-map either — ask: "Which CI check do you want to fix? (type the check name)"

## Step 4 — Fetch failing CI logs

```bash
PR=$(gh pr view --json number --jq '.number')
RUN_ID=$(gh pr checks $PR --json name,link --jq '.[] | select(.name=="{check-name}") | .link | split("/runs/")[1] | split("/")[0]')
gh run view $RUN_ID --log-failed
```

If the check is currently passing — report: "`{check-name}` is passing. Nothing to fix." and stop.

If `RUN_ID` is empty — the check name wasn't found in the PR checks. Show available check names:

```bash
gh pr checks $PR --json name --jq '.[].name'
```

Stop and report: "Check `{check-name}` not found. Available checks: {list}"

Read the log output carefully to understand what is failing before proceeding.

## Step 5 — Reproduce and fix

Activate `nico-dev:scope` skill.

Determine the fix approach based on resolved `type`:

### type: lint

Activate `linting:linting_fix` skill.

```bash
{fix-command}
```

### type: svelte

Activate `nico-dev:svelte5-developer` skill.

```bash
{fix-command}
```

Read errors carefully — common categories: type errors in `.svelte` files, missing props, import failures.
Work through each error. Read affected files before editing.

### type: test

Activate `nico-dev:coding` skill.

```bash
{fix-command}
```

Read failing test names and messages from the CI log. Reproduce the same failures locally first.
Fix source code (or test if the test itself is wrong), not just the symptom.

### type: unknown / no ci-map

Report: "No local fix command configured for this check. Review the CI log above and fix manually."
Show the log output. Stop here.

## Step 6 — Verify

```bash
{verify-command}
```

If errors remain — analyze the output and repeat the fix. Continue until clean.

## Step 7 — Report

Show what was fixed (files changed, errors resolved).
Stop here — do not commit.
