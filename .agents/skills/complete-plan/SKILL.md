---
name: complete-plan
description: Turn a finished plan into documentation and delete it. Use when a plan's last task is done and verified, when its remaining items have all shipped, or when asked to close, retire, land or convert a plan into docs.
---

# Complete a plan

A plan lives in `PLAN_DIR` only while work remains. When it is done, its durable
content moves to the docs that own the behavior and the plan is deleted.

Resolve two per-repository parameters from the repository's own documentation
before starting; ask only when neither is documented:

- `PLAN_DIR` — where plans live.
- `DOCS_ROOT` — the folder whose documentation owns behavior, and its index file.

## 1. Confirm it is done

Check the code and tests, not the plan's own checkboxes: each claimed item has
source, a test or a receipt on the main line. Items still open mean the plan is
only partly done: keep it, trimmed to what is left (step 4).

## 2. Pick the destination

Choose the doc that already owns the behavior:

- One package or feature: its `README.md`, or its own `docs/<topic>.md` when the
  design, research or evidence is too long for the README.
- Cross-cutting behavior (testing, workflow, repository conventions):
  `DOCS_ROOT`, with the file added to that folder's index.

Read the destination first and write in its voice: plain sentences, present
tense, the behavior as it is now.

## 3. Move what lasts, drop the rest

Keep:

- the decisions and why they were made, including options rejected for a
  reason that still holds;
- constraints and limits, and what is deliberately not done;
- the final behavior, how to use it, how to verify it (exact commands);
- benchmark conclusions and measured numbers that justify a default.

Drop: task checklists, progress tables, "Resume here" notes, session or
workspace ids, revision hashes of work in progress, dated status lines, and
alternatives that no longer matter.

Evidence files the docs link to (benchmark results, source snapshots,
receipts) stay or move next to the doc; never leave a doc pointing into
`PLAN_DIR`.

## 4. Delete or trim the plan

- Done: delete the plan file or folder.
- Partly done: rewrite it to the open items only, with a one-paragraph header
  linking to where the finished part is documented.
- Update `PLAN_DIR`'s index when it has one, and fix every link to the plan
  (`grep -rn '<plan-name>' --include='*.md' .`).

## 5. Land it

One revision per plan (or per topic when plans overlap), e.g.
`docs(<scope>): fold the <plan> into <destination>`, with a body saying what
moved and what was dropped. Run the repository's format and lint checks.
