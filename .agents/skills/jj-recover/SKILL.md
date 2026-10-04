---
name: jj-recover
description: >
  Diagnose and recover from Jujutsu errors or inconsistent history and workspace
  state. Use when a jj command fails, conflicts or stale workspaces appear,
  change IDs diverge, or a prior history operation needs recovery. Also load when
  a history-shape request — "unify the history", "spine branch", "linear
  history", "one branch", "too many heads" — already failed or forked the line.
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
- **Never reuse a commit-id captured before a rewrite.** Hole-filling rewrites
  descendants (`Rebased N descendant commits onto updated working copy`), so a
  planned id is stale the moment an earlier step lands. Resolve by change-id at
  use time, or duplicate the original by id —
  `jj duplicate -r <original-id> -o <tip>` — which neither races nor stales.
- **Never run `jj squash` or `jj describe` without `-m` or
  `--use-destination-message` from an agent shell.** Both open an editor and hang
  until the job is killed.
- **Never `jj describe`, `jj squash` or `jj abandon` a revision another
  workspace's `@` sits on.** That workspace's next snapshot turns it into a
  divergent copy of the same change (same change-id, second commit-id); name only
  revisions you created.
- **Never `jj op revert` an operation you already reverted.** Reverting a revert
  re-applies the original effect, so a chain of reverts lands somewhere you did not
  predict. `jj op revert` takes **one** operation, so a loop must run newest-first;
  once the window is yours, prefer a single `jj op restore <good-op>` (it discards
  other agents' operations too — say so in the report).
- **Never move an interior revision to the tip with `jj rebase -r <rev> -d <tip>`.**
  Its descendants are re-applied onto its old parent, which re-introduces conflicts
  you already resolved. A workspace `@` that sits mid-line moves with
  `jj new <tip>` **inside that workspace**.

## Before rewriting history

1. Note the current `jj op log -n 1` id. `jj op restore <id>` restores the
   repository to the state **after** that operation, not before it.
2. Avoid concurrent writes to a workspace; one workspace can rewrite another's
   working copy and cause "Concurrent modification detected".
3. Snapshot important tree invariants and re-check after each rewrite.
4. Snapshot every workspace first — run `jj status` inside each one. Edits that
   were never snapshotted are not in the store, and a rewrite of that workspace's
   `@` is what races them.
5. Read a past state without touching the present with `jj --at-op <op-id> log …`.
   It is the cheapest way to prove a prune changed no tree: capture the tip before
   the op, then `git diff <old-tip> <new-tip>` = 0 files.

## Conflict checks that actually work

Per-commit template scan over explicit ranges:

```bash
jj log -r '<range>' --no-graph -T 'if(conflict,"C",".\n")' | grep -c '^C$'
jj log -r '<range> & heads(all())' --no-graph -T 'if(conflict,"C ","") ++ description.first_line() ++ "\n"'
```

Hidden commits break revset algebra — enumerate by explicit id or range endpoints.

One unresolved conflict marks the commit **and its whole subtree**, so a conflict
count is not a location. Walk the line from the base and take the first
conflicted commit; that is the root. Resolve it, re-scan, repeat.

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
- `jj op revert <op-id>` undoes exactly one operation; `jj op restore <op-id>`
  reverts everything after it. In a shared store, revert your own ops one by one
  rather than restoring over other agents' work.
- `jj resolve -r <rev> --tool :ours|:theirs` resolves a non-`@` revision and
  rewrites it (bookmarks follow). Without `--tool`, `-r` only prints the
  `jj new` + resolve + squash instructions.
- Abandoning a workspace's `@` gives that workspace a fresh empty commit on the
  nearest **visible** ancestor, which can sit below the branch you pruned. Sweep
  `jj workspace list` afterwards and rebase the strays onto the tip.
- Abandon a whole junk set in one operation:
  `jj abandon '<base>:: & ~::<tip>'`. It prints `Deleted bookmarks:` for every
  bookmark that pointed into the set; re-create the ones you still want.
- `jj file show` reporting "exists but is not a file" means the path is a symlink
  or gitlink in the tree. Read the mode with `git ls-tree <rev> <dir>`
  (120000 symlink, 160000 submodule).
- Rewriting the tip orphans every live `@` that was its child (new commit-id, or a
  divergent copy of the same change). Fold them back with
  `jj rebase -r <stray-head> -d <tip>` and abandon the duplicate copy. Expect to
  repeat while other agents keep committing.
- `jj log -T 'commit_id'` over a multi-revision set concatenates ids with no
  separator; always template `commit_id ++ "\n"`.
- `jj bookmark set` refuses a backwards or sideways move while the bookmark is
  conflicted; choose the target first, then pass `--allow-backwards`.
- In a colocated repo, a `jj workspace add` checkout has no `.git`: git commands
  there need `GIT_DIR=<root>/.git GIT_WORK_TREE=<ws>`, and `git apply --3way`
  fails against the shared index.
- Two copies of one change that differ only by `.jjconflict-*` side files are a
  materialised conflict, not two implementations. `jj rebase` drops the inherited
  side files because they cancel in the change diff; a git patch of the same
  commit re-adds them.
