---
name: jj-merge-workspaces
description: >
  Consolidate completed work across Jujutsu workspaces into a linear history
  and leave the default workspace at its tip. Use when the user asks to unify
  workspace work, bring completed changes into the default workspace, or create
  one linear history from parallel workspace branches.
---

# Merge jj workspace work

## Interpreting requests

When the user asks to unify workspace work or bring completed changes into the
default workspace, treat that as a direct request for this workflow. Identify
the target bookmark (use `main` only when that is the repository's intended
target), relevant completed workspace heads, and active work. Preserve unrelated
work and do not rewrite a workspace while its task is active. Do not abandon,
complete, or forget another workspace's work as a cleanup shortcut.

The desired result is one linear history at the target bookmark, with the
default workspace working copy left as a new empty child of the unified tip.
After moving the bookmark, run `jj new <target-bookmark>` in the default
workspace; running it in a sibling workspace does not update the default one.

## Why duplicate first

jj refuses to rebase commits owned by another workspace ("stale workspace"
error). Workaround: duplicate those commits into your workspace, rebase the
copies into one chain, move `main` to the end, then start a new change.

## Steps

1. Map the workspaces:
   - `jj workspace list`
   - `jj log --ignore-working-copy -r 'all()' --no-graph`
   - Confirm which workspace is the default, which heads are complete, and that
     no selected workspace is still active.

2. Duplicate commits you don't own (skip empty working copies):
   - `jj duplicate <start>::<end>` — e.g. `jj duplicate oysxvorl::munqmoxq`
   - Note the new change IDs it prints; use them in the rebases.

3. Rebase into a linear chain:
   - `jj rebase -b <copied-tip> -o <destination>` — `-b` moves the whole
     copied chain (the tip plus its ancestors that aren't already in the
     destination). If the copies' fork base isn't in the destination's
     ancestry, use `jj rebase -s <first-copy> -o <destination>` so the
     originals aren't dragged along.

4. Resolve conflicts:
   - `jj resolve --list` to find conflicted files
   - `jj new <conflicted-commit>`, fix the files, then
     `jj squash --use-destination-message` (plain `jj squash` opens an editor)
   - Descendant conflicts often auto-resolve after the squash.

5. Move main and continue:
   - `jj bookmark set <target-bookmark> -r <final-commit>`
   - In the default workspace: `jj new <target-bookmark>`

## Consolidating an exploded history

When dozens of heads must collapse into one line (a previous unification or a
`jj duplicate` burst), separate live work from junk before rewriting anything:

```bash
BASE=<base-revision>
WS=$(jj workspace list | awk -F'[ :]+' '{print "::"$4}' | paste -sd'|' -)
jj log -r "$BASE:: & ($WS)"   # keep: ancestors of a live workspace @
jj log -r "$BASE:: & ~($WS)"  # junk: stale copies, abandoned variants
```

- Rebuild with `jj duplicate -r <original-id> -o <tip>` per kept revision.
  Duplicating a range and rebasing `-b` afterwards goes stale as soon as the
  first member is rewritten, because hole-filling rewrites descendants.
- `-r` keeps only the edges **inside** the set: `jj rebase -r 'a|b|c' -d <tip>`
  chains a real parent-child run, while an unrelated set becomes siblings, not a
  line.
- Two heads implementing the same change with different content cannot both sit
  on one line coherently: pick a side (`jj resolve -r <rev> --tool :ours|:theirs`)
  or drop the duplicate. Union only when the sides added distinct content.
- Abandon the junk in one operation, `jj abandon "$BASE:: & ~::<tip>"`, then
  sweep `jj workspace list`: abandoned workspace `@`s are re-created on the
  nearest visible ancestor, which can be below the base revision.
- Verify with the repository's own gates (install, typecheck, build, test), and
  intersect the failing paths with `jj diff --from <pre-merge> --to @ --summary`
  to separate baseline drift from damage this rewrite caused.

### Avoid

- **Linearizing the junk set.** Two variants of one change merge into
  non-overlapping hunks: the files look resolved and are still broken.
- **Rebasing a list of commit ids captured earlier.** Hole-filling rewrites
  descendants, so the next id is already stale.
- **Unioning a conflict between two implementations of the same change.** Pick a
  side or drop the duplicate.
- **Reading a conflict count as a location.** One root conflict marks every
  descendant.
- **Writing files while the checkout's `@` is mid-line.** Any write amends that
  revision and rebases the subtree under other agents.
- **Bare `jj squash` / `jj describe`.** They open an editor and hang; pass `-m`
  or `--use-destination-message`.
- **`jj op restore` in a shared store.** It discards other agents' operations
  too; use `jj op revert` on your own, newest first.
- **Declaring success from shape alone.** Run the gates.

## Example

```bash
jj rebase -b xoqntywy -o main              # commits you own
jj duplicate mkllukrx::ltullnxy            # chain from another workspace
jj duplicate oysxvorl::munqmoxq
jj rebase -b <dup1-tip> -o tqqytsoq        # chain the copies in order
jj rebase -b <dup2-tip> -o <dup1-tip>
jj bookmark set main -r <last>          # use the intended target bookmark
jj new main                             # run in the default workspace
```

## Gotchas

- Stale workspace error -> duplicate the commits first, then rebase the copies.
- `-b` is relative to the destination: it never re-moves what's already there.
- Conflict with valid code on both sides -> merge both sides, don't drop one.
- If a rebase or conflict repair is unclear, stop and use `jj-recover` to
  diagnose before making more history changes.
- Verify the final chain: `jj log --ignore-working-copy -r '<target>::@' --no-graph`
