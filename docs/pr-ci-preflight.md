# Feature: Local CI Preflight in `create-auto`

> **Status:** Planning
> **Last Updated:** 2026-06-10

---

## Summary

Run CI checks locally before pushing, so remote CI failures are caught before the roundtrip. The `create-auto` pipeline already handles branch → commit → push → PR; this feature inserts a CI preflight step between commit and push.

---

## Design

### Discovery (Step 2)

Add to the parallel inspection block:

```bash
find .github/workflows -maxdepth 1 \( -name "*.yml" -o -name "*.yaml" \) 2>/dev/null
```

### Command extraction (Step 5)

Build `$CI_COMMANDS` from the discovered files:

1. Read each workflow file
2. Keep only workflows with `pull_request:` in their `on:` section
3. Collect `run:` values — skip steps that have `uses:`, and skip infrastructure lines: `corepack enable`, `yarn install`, `yarn install --immutable`, `npm ci`, `npm install`, `pip install`
4. Deduplicate identical commands (first occurrence, preserve order)
5. Lint substitution: if `yarn lint` or `npm run lint` is present, check `package.json` for a `lint:changed` script; if found, substitute

### Plan preview (Step 6)

```
  [3. CI preflight:   {$CI_COMMANDS joined with " + "}]   ← only if push planned and $CI_COMMANDS non-empty
```

### Execution (Step 7, between Commit and Push)

For each command in `$CI_COMMANDS`:

- Report `⏳ {command}`
- On success: `✅ {command}`
- On failure: `❌ CI failed: {command}` → print last 30 lines → **BLOCK**

On failure: stop, show output, wait for user to fix, then **re-run all `$CI_COMMANDS` from the start** before pushing.

---

## Tested against

`chessarms/tsmain` — `code-quality.yml` runs three PR jobs: lint, svelte-check, test.

Resolved command list after extraction and dedup:
1. `yarn lint:changed` (substituted from `yarn lint`)
2. `yarn build:all`
3. `yarn workspace webapp run check`
4. `yarn test:unit`

---

## Known issues (from code review)

| # | Severity | Issue |
|---|----------|-------|
| 1 | High | **Denylist is incomplete.** `apt-get`, `docker build`, `./scripts/setup.sh`, and other infrastructure `run:` steps pass through and execute locally. Fix: flip to allowlist — only collect commands whose first token is `yarn`, `npm`, `npx`, `pnpm`, or `node`. |
| 2 | Medium | **Empty `$CI_COMMANDS` renders a blank step in the plan.** Add guard: suppress the CI preflight line when `$CI_COMMANDS` is empty. |
| 3 | Medium | **"PR only" path bypasses CI entirely.** When the user already pushed manually, `create-auto` routes to "PR only" and never populates `$CI_COMMANDS`. Consider running preflight whenever a PR is about to be opened, not only when push is in the plan. |
| 4 | Medium | **No re-run protocol after a fix.** Current spec says "wait for user to fix" but doesn't specify re-running CI. Must explicitly re-run all `$CI_COMMANDS` from the start before proceeding to push. |
| 5 | Low | **Lint substitution misses `pnpm run lint`.** Only `yarn lint` and `npm run lint` are covered. Add `pnpm run lint` / `pnpm lint`. |
| 6 | Low | **Lint substitution match is ambiguous.** "If `yarn lint` is in the list" could be read as substring match, incorrectly substituting `yarn lint:packages`. Clarify as exact-element match. |

---

## Next steps

- [ ] Fix issue #1 (allowlist approach for command filtering)
- [ ] Fix issues #2–#4 (empty guard, PR-only gap, re-run protocol)
- [ ] Fix issues #5–#6 (pnpm + exact match lint substitution)
- [ ] Implement in `plugins/git/commands/pr/create-auto.md`
- [ ] Test on `chessarms/tsmain`
