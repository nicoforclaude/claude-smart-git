---
description: Merge dev-preview into main branch to catch up with latest changes
---


## Goal 
The goal is to keep main branch (local) up to date with other branches.
That happens when user accidentially works, for example in `dev-preview` while fixing some deploy bugs and main is left behind.

## Conditions

Main node should be "parent" (say otherwise) of commit node that is ahead and user currently in.

## Implementation


For that to happen, no pending changes should be in git.

Steps:
1. Verify no pending changes
2. Checkout main branch (locally)
3. Merge dev-preview into main
4. Return to original branch

Use this when dev-preview has been tested and is ready to be promoted to main locally.
