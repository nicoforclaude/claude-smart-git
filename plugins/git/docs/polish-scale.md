# Polish Scale (0–10)

Review time is valuable: prod backend deserves much polish, deep integration tests little. The polish level says how far review findings get fixed before a PR counts as ready to merge.

Polish is separate from whether fixing happens at all: a PR is either left as is, or made ready to merge at a polish level. Ready to merge at polish 0 still means green CI — it just skips the review findings.

`/git:pr:create-auto` recommends a level; the user confirms or overrides it in the Go/No Go dialog.

## Scale

Each anchor adds one kind of finding to the fix list; a level includes everything below it.

| Polish | Name | Adds to the fix list | Typical use |
|--------|------|----------------------|-------------|
| 0 | Skip | Nothing from the review — only what blocks the merge (red CI) | spikes, throwaway |
| 2 | Light | What breaks: crashes, data loss, security holes | dev pages, scripts |
| 4 | Fair | What is wrong in some case: edge cases, error paths, a test asserting the wrong thing | admin tools, integration tests |
| 6 | Solid | What will cost later: convention breaks, misleading names, duplication, a missing obvious test | features, UI |
| 8 | Thorough | What a careful teammate would raise: structure, readability, thin test coverage | prod backend |
| 10 | Hard | Everything, until the review has nothing left to say | shared core, public API |

Odd numbers lean between neighbours: 3 is Light plus the cheap Fair items.

## Rule

Fix what the level includes, dismiss the rest with a one-line why, and stop when a pass turns up nothing the level includes.

There is no round limit — the rule ends the loop.

## Recommending a level

Judge from the changed file paths:

1. **Project map** — if the project's CLAUDE.md maps areas to polish levels, use it
2. **Otherwise** — match the paths against the "Typical use" column
3. **Mixed PR** — take the level of the area holding most of the change; lean up when a higher-level area is touched non-trivially

State the recommendation with a one-line reason: `4 Fair — 9 of 10 changed files are integration tests`.
