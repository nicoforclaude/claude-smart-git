# Handling 🚫 Local-Only Files

When analyzer detects 🚫 files, fix before proceeding with user's commit.

## Steps

1. Add pattern to `.gitignore`
2. Untrack if already tracked: `git rm --cached <file>`
3. Commit: `chore(git): ignore local config files`

## Priority

Handle as **first commit** (Priority 0) before user's actual work.

## Note

`git rm --cached` removes from tracking but keeps the local file intact.
