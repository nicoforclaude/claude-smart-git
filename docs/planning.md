# Planning Notes

## Handling Problematic Files in Git Diff

When detected in git diff output:

| Pattern | Recommendation |
|---------|----------------|
| `.claude/settings.local.json` | Suggest adding to `.gitignore`, do not commit |
| `nul` files (Windows null device) | Suggest deleting, add to `.gitignore` |

