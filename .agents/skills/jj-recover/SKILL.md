---
name: jj-recover
description: >
  Diagnose and recover from Jujutsu errors or inconsistent history and workspace
  state. Use when a jj command fails, conflicts or stale workspaces appear,
  change IDs diverge, or a prior history operation needs recovery.
---

# Recover from jj history and workspace problems

## Recovery workflow

1. Stop repeating the failing mutation; capture the exact error.
2. Inspect `jj status`, `jj workspace list`, `jj op log -n 1`, and relevant commits.
3. Identify workspace owners and active work. Preserve other workspaces; do not
   abandon or rewrite their work without an explicit request.
4. Diagnose read-only first. Make the smallest repair, then verify conflicts,
   trees, bookmarks, and working copies.

## Never do this

- **Never target destructive ops by change-id.** Divergent change-ids auto-resolve
  to an arbitrary visible copy — you can kill your own ancestor while aiming at a
  duplicate. Always resolve to a **commit-id** first.
- **Never trust `(range) & conflict` as a cleanliness check.** It silently returns
  empty for hidden commits. Conflicted commits inside the ancestry will read as "0".
- **Never `jj restore --from <rev> <path>` blindly** — path args are fileset
  patterns; `$` in paths (e.g. `o.$orgSlug`) is a syntax error, and a failed
  lookup still truncates the redirect target to an empty file.

## Before rewriting history

1. Note the current `jj op log -n 1` id. `jj op restore <id>` restores the
   repository to the state **after** that operation, not before it.
2. Avoid concurrent writes to a workspace; one workspace can rewrite another's
   working copy and cause "Concurrent modification detected".
3. Snapshot important tree invariants and re-check after each rewrite.

## Conflict checks that actually work

Per-commit template scan over explicit ranges:

```bash
jj log -r '<range>' --no-graph -T 'if(conflict,"C",".\n")' | grep -c '^C$'
jj log -r '<range> & heads(all())' --no-graph -T 'if(conflict,"C ","") ++ description.first_line() ++ "\n"'
```

Hidden commits break revset algebra — enumerate by explicit id or range endpoints.

## Fixing conflicted commits

Oldest first — one deep fix often cascade-heals descendants:

```bash
jj new <conflicted-commit-id>        # by commit-id
# resolve markers (union both sides unless one side is strictly newer)
jj squash --into <commit-id> --use-destination-message
```

Re-scan after each; repeat until clean.

## Linearizing an N-parent merge

Keep one side as the spine; never hand-resolve inside the rewritten merge:

```bash
jj rebase -s <branchB-unique-start> -d <spine-tip>   # per parallel branch, oldest content first
jj rebase -s <post-merge-line> -d <new-tip>          # move descendants over
jj diff --from <new-tip> --to <original-merge-id> --stat   # MUST be empty before next step
jj abandon <original-merge-commit-id>
```

Keep the original merge commit-id as **golden reference**: resolve every cascade
conflict with `jj restore --from <golden> <paths>` + squash into the owning commit.
The empty-diff gate is mandatory — rebasing across reformatted regions can silently
drop whole blocks with zero conflict markers (only tree-equality detects it).

## Gotchas

- `jj op restore <op>` restores **to** that state, including its effects.
- Stale working copies after cross-workspace rewrites: re-materialize with
  `jj new <tip>` inside the affected workspace.
- `$`-paths: `jj` treats path arguments as fileset expressions. Escape the path
  for jj's fileset syntax; don't pass a bare path to `jj file show`.
- Empty `wip` heads multiply from workspace churn; sweep with a fixpoint loop
  excluding only the live branch ancestry.
- Commit-ids go stale after every cascading rebase — fetch the target id
  immediately before each squash/abandon; squashing into a superseded copy is a
  silent no-op. Divergent change-ids also break revsets (`x & ::tip` errors);
  disambiguate with `change_id(x) & ::tip` first.
- Abandoning only a junk head exposes its parent as a new head — abandon the
  whole orphan chain (`::<junk-head> & ~::<fork-point>`, by explicit commit ids).
- Before pruning, prove a candidate is outside the kept ancestry; abandoning a
  head can expose its parent as another stray.
- `jj abandon` silently deletes bookmarks dangling on junk ("Deleted bookmarks:"
  line). Recover what they pointed at via `jj op show <abandon-op>`, re-point at
  the kept counterpart of the same change-id.
