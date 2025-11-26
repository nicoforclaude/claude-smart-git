# Handling 🚫 Files (Local-Only and Suspicious)

When analyzer detects 🚫 files, fix before proceeding with user's commit.

## Steps

1. Show user which files were flagged and why
2. Suggest pattern to add to `.gitignore`
3. If user agrees: add pattern, untrack if needed (`git rm --cached <file>`), commit: `chore(git): ignore local/sensitive files`

## Priority

Handle as **first commit** (Priority 0) before user's actual work.

## Note

`git rm --cached` removes from tracking but keeps the local file intact.
