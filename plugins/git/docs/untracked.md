# Local-Only Files (🚫)

Files that should stay local but never be committed.

## Patterns

- `settings.local.json` - User-specific Claude Code settings
- `*.local.json` - Any local JSON config
- `.env.local` - Local environment overrides

## Detection

When these appear in `git status` (staged or untracked), flag as 🚫 Local Only.
