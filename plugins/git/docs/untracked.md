# Local-Only and Suspicious Files (🚫)

Files that should stay local but never be committed.

## Explicit Patterns (always flag)

- `settings.local.json` - User-specific Claude Code settings
- `*.local.json` - Any local JSON config
- `.env.local` - Local environment overrides

## Suspicious Patterns (suggest gitignore)

- `.local/` directories - local data/cache
- `errors/`, `logs/`, `debug/` directories with debug output
- `.env*` files (except `.env.example`)
- `*secret*`, `*credentials*`, `*password*`, `*token*` in filename
- `*.log`, `*.bak`, `*.backup` files
- IDE/editor folders: `.idea/`, `.vscode/settings.json`, `*.code-workspace`

## Detection

When these appear in `git status` (staged or untracked), flag as 🚫 and suggest adding to `.gitignore`.
