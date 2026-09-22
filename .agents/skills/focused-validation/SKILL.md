---
name: focused-validation
description: Choose proportionate checks, reproduce failures first, and report focused proof separately from baseline or broad-suite failures.
---

# Focused validation

Use after a change, while debugging a failure, or when a repository has a
large release gate that would obscure useful feedback.

## Choose the smallest useful proof

1. Reproduce the reported behavior or failure with the narrowest real path.
2. Run the single test, package check, browser scenario, query plan, or build
   lane that proves the changed boundary.
3. Widen only when the change crosses a shared package, generated contract,
   runtime matrix, or release boundary.
4. Leave the broad release gate to CI or an intentional release verification.

Prefer real implementations at the boundary under test. Use mocks only where
the repository contract makes them the explicit seam; do not replace a real
database, provider, browser, or generated consumer with a tautological stub.

## Evidence

Report:

- the exact focused command and its result;
- the behavior or contract it proves;
- broader checks that were intentionally skipped;
- known baseline failures, if any; and
- remaining runtime or deployment caveats.

Do not call a change complete because a formatter passed, a type assigned, or a
static scanner produced no finding when the user-visible behavior was not
exercised.

## Debugging discipline

Use a single-file or single-scenario loop while iterating. Keep complete noisy
diagnostics in an artifact when needed, but surface failure-first evidence in
the handoff. Add a regression check for a repaired behavior when the contract
could break again.
