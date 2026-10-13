---
name: maintain-verification-skill
description: Keep a project's verification skill (verify-<app>), its CLI and its feature map in sync with the app after a change that affects how the app is launched, seeded, driven or verified. Use after changing dev/e2e fixtures, the verification CLI, seed data, auth for tests, the feature catalog schema, or a feature's user path; or when a verification run failed because the skill or map was wrong.
---

# Maintain a verification skill

Event-driven, not periodic: run it as the last step of a change that touched
the verification process, scoped to what that change affected.

## Trigger check

Run it when the change touched any of:

- launch, seed, login or teardown code, or the verification CLI;
- e2e fixtures or helpers the CLI wraps;
- the feature catalog or map format;
- a feature's user path (renamed menu, moved button, new route or flag).

If none applies, stop.

## Process

1. **Hygiene.** Every feature map entry resolves (links, catalog ids); no
   duplicates; no feature that no longer exists; new user-facing features
   from this change have an entry.
2. **Source wave (read-only).** For each affected feature, re-read the code
   that defines its entry point and user path. Note where the map, the CLI or
   the skill text disagree with the code. Parallelize per feature when there
   are several.
3. **Reconcile.** Update the map entries, the CLI and the skill text to match
   the code. The code is the source of truth for paths; the skill is the
   source of truth for how to drive them.
4. **Live pass (required).** Using the verification CLI: up → doctor → open
   each affected feature → evidence → down. A map entry not driven live this
   session stays unverified. Static checks only catch syntax: a wrong label
   or moved button passes them and fails here.
5. **Triage each failure.**
   - Doc or map drift: fix it.
   - Harness gap (CLI cannot reach or capture something): fix the CLI.
   - Product regression (the app is wrong): report it with evidence; never
     change product code or weaken the check to make the pass green.
   - Environment (load, network, credentials): retry once, then report.

## Outcome

End with exactly one:

- **Clean:** nothing drifted; live pass green.
- **Changed:** what was fixed, as one described revision/commit, plus the
  green live pass.
- **Blocked:** what failed, evidence paths, and whether it is a product
  regression or an environment problem.
