---
description: "Live monitor for a just-pushed PR — polls CI checks and the automated review comment, fail-fast on red checks"
argument-hint: "[PR number]"
allowed-tools: Bash, Read, AskUserQuestion, Skill(windows-shell:windows-shell)
---

# PR: Watch — Live monitor (CI + review)

Watch a PR after push: report the automated review comment when it lands, or the first failing CI check — whichever comes first. Do not end the turn with a silent "done" — the watch keeps the loop closed.

Needs the PR number (from `$ARGUMENTS`, the invoking command, or `gh pr view --json number`).

## Poll loop

Run in background — every 60 seconds, up to 10 minutes, each round checks CI first, then the review comment:

```bash
PR={number}; PUSHED_AT=$(date -u +%Y-%m-%dT%H:%M:%SZ)
for i in $(seq 1 10); do
  sleep 60
  fail=$(gh pr checks $PR --json name,bucket,link \
    --jq '[.[] | select(.bucket == "fail")] | first // empty | "\(.name) \(.link)"' 2>/dev/null)
  if [ -n "$fail" ]; then echo "CHECK_FAILED $fail"; exit 0; fi
  url=$(gh pr view $PR --json comments --jq \
    "[.comments[] | select(.author.login == \"github-actions\" and (.body | startswith(\"## Review\")) and .createdAt > \"$PUSHED_AT\")] | last | .url")
  if [ -n "$url" ] && [ "$url" != "null" ]; then echo "REVIEW_READY $url"; exit 0; fi
done
echo "REVIEW_TIMEOUT"
```

- **Fail-fast:** the loop exits on the first red check — it never waits out the window. If the failed check is the review job itself, no review comment is coming, so exiting immediately is correct.
- The `createdAt > $PUSHED_AT` filter matters: it skips review comments from earlier pushes.

## On `CHECK_FAILED` — classify before remediating

**Never fix blindly — read the job log first:**

```bash
gh run view {run-id} --log-failed
```

(Get the run id from the check's link, or `gh pr checks $PR` output.)

Then classify:

**Auth failure in the review job** (check name suggests claude/code-review AND the log shows an authentication/credentials error) — suspect an expired `CLAUDE_CODE_OAUTH_TOKEN`. Suggest remediation to the user, in this order — **never auto-run secret updates; they stay behind an explicit go:**

1. A repo-local command in `.claude/commands` that refreshes the secret and reruns the job — look for a CI/secrets-related command (e.g. a restart-code-review-job command)
2. If available, `/root:secrets:update_claude_secrets` run from the workspace root, then rerun the failed job:
   ```bash
   gh run rerun {run-id} --failed
   ```

**Any other failure** (lint, tests, build) — normal fix-the-code territory. Report the check name and a short excerpt of the failing log, and suggest `/git:pr:fix-ci` to reproduce and fix locally.

## On `REVIEW_READY`

Fetch the comment body and report:

1. The review's issues table (or a condensed version if long)
2. Per item, a **reaction verdict**:
   - `needs reaction` — real defect, phantom dependency, broken behavior → propose the fix
   - `decision` — reviewer flags a tradeoff; user must choose → state the options in one line
   - `no action` — stale-comment nits already acceptable, intentional-state items, false positives → say why in one line
3. If any item needs reaction, offer to fix on the same branch now (do not fix without a go)

## On `REVIEW_TIMEOUT`

Report that no review appeared within ~10 minutes and give the manual check commands:

```bash
gh pr checks {number}
gh pr view {number} --comments
```
