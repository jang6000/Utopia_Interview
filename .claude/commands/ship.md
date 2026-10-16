Run the shipping workflow end to end. Follow these phases in order and stop if any phase fails.

This is the project-agnostic version. If the current project has its own `.claude/commands/ship.md`, that one wins — it will know the real deploy target, rules files, and verify hook.

## Phase 0 — Review (conditional)
Run `/code-review` (Opus) on the unpushed diff **before** lint/build. Logic bugs that lint and build pass through — import shadowing, wrong hook deps, "runs but does the wrong thing" — get caught here or not at all.
```
git diff --name-only origin/main..HEAD
```
- **Skip** only if every change is a typo/copy edit, a dependency version bump, or a pure bug fix with no logic change. (Same skip rule as the Docs phase.)
- Otherwise run the review, then **fix every correctness finding before proceeding.** Do not ship over an unresolved correctness finding.
- Non-correctness suggestions (style, simplification) are optional — use judgment.

## Phase 1 — Docs (conditional)
Any design, UX, or architecture decision in this batch must already be documented — same session, not retroactively.
- A named entry in the project's domain doc (`CONTEXT.md` or equivalent), including an `_Avoid_` anti-pattern.
- An ADR in `docs/adr/NNNN-slug.md` for every non-obvious or hard-to-reverse decision: why this approach, what was rejected, what the trade-off is. Increment from the last file.
- **Client handover pack**, if the project keeps one (its `CLAUDE.md` names the file and what triggers an update). Check the batch against those triggers — anything that changes what the client's staff do, what a visitor sees, a stated limit, or an account/cost — and update the pack in plain client language. It goes stale silently otherwise.
- **Skip** only for pure bug fixes, typo/copy changes, dependency bumps.

If decisions went undocumented, write them now before shipping. Undocumented decisions become invisible debt.

## Phase 1b — Knowledge graph refresh (conditional)
**Only if `graphify-out/graph.json` exists in this project.** If it does not, skip silently — do not build one during a ship.

The graph is documentation, so it refreshes with the docs, before anything is committed:
```
/graphify . --update
```
`--update` re-extracts only the files that changed since the manifest, against the cache. What that costs depends on what changed:
- **Code files only** — AST extraction, no LLM, effectively free. Let it run.
- **Docs, schema, or images changed** — semantic subagents run. Still cheap on a normal batch, since untouched files come from cache.

Then commit the refreshed graph **with the docs group** in Phase 4, not as its own commit. Whichever graph files the project tracks are the ones that ship; the rest of `graphify-out/` is normally gitignored machine state and must not be force-added.

If the update errors, say so and continue — a stale graph is not a reason to block a ship. Note it in the final summary so it gets rebuilt later.

## Phase 2 — Pre-Ship Checklist
Run the project's linter (`npm run lint`, `ruff`, `golangci-lint`, whatever applies):
- If lint has errors: report them and stop. Do not proceed.
- Warnings are okay, errors are not.
- **Build:** do NOT run the build automatically — the user verifies the build locally. Before the Phase 4 push, confirm with the user that the local build passes. If unconfirmed, stop and ask. (This is the documented `no-auto-build` preference; `/ship` does not override it.)
- To check correctness without a full build, prefer a typecheck (`npx tsc --noEmit`, `mypy`, etc.) plus the linter.

### Generated changelogs — check for a commit hook first
Some projects auto-generate changelog entries from commit subjects via a `post-commit` hook. **Before running any changelog script manually, check `.husky/` or `.git/hooks/` for one.** Running both a hook and a manual script produces duplicate entries and extra version bumps.

If such a hook exists:
- Do NOT run the changelog script for commit types the hook handles (usually `feat`/`fix`).
- After commits land, review the generated entries — auto-entries use the commit subject verbatim and read tersely. Enrich them, delete duplicates, and fold the edit into the batch's `docs:` commit (hooks typically skip `docs:`/`chore:`).
- **Assume generated changelogs are user-facing.** Write commit subjects and bodies in plain language, no raw dev jargon — they land verbatim in front of end users.

## Phase 3 — Infra / rules deploy (conditional)
Check whether any infrastructure or security-rules files changed across ALL unpushed commits (a batch is usually several commits, so `HEAD~1` is not enough):
```
git diff --name-only origin/main..HEAD
```
If security rules, IaC, migrations, or schema files appear, deploy them with the project's documented command before the code push, e.g.:
```
firebase deploy --only firestore:rules
firebase deploy --only storage
```
If nothing infra-related changed, skip this phase silently.

## Phase 4 — Push
Commit in categorized batches (file-level grouping by feature, dependency order: foundation/security → infra → features → docs), then a **single push** at the end.
```
git push origin main
```

## Phase 5 — Verify
If the project exposes a deployment record or status hook, query it and confirm the deploy succeeded. If the project has no queryable deploy (self-hosted, teammate-managed, or handed off), say so explicitly rather than implying the push reached production.

**Abort and roll back** if the build fails or an infra deploy errors:
```
git revert HEAD && git push origin main
```

Report a one-line summary at the end: what shipped, the version if the project tracks one, and where it actually landed.
