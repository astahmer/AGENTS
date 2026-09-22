---
name: frontend-evidence-audit
description: Audit frontend quality with machine findings, runtime browser evidence, leverage-based prioritization, and self-contained plans.
---

# Frontend evidence audit

Use when auditing a React or browser application for bugs, performance,
accessibility, security, or maintainability rather than fixing one known line.

## Recon first

- Identify the framework, rendering mode, state/data libraries, styling system,
  authentication path, and shipped hot paths.
- Run the repository's read-only static scanner or analyzer and preserve its
  structured report.
- Capture runtime evidence when behavior matters: DOM, screenshots, console,
  network, computed styles, accessibility tree, or profiler data.

## Vet findings

Re-read every cited location. Reject findings that are deliberate, duplicated,
out of scope, or analyzer noise. Severity is leverage, not scanner severity:

- high: user-visible bugs, unsafe input sinks, whole-tree rerenders, or broken
  primary interactions;
- medium: bounded correctness, performance, or accessibility problems;
- low: polish, dead code, or cold-path maintainability issues.

Keep missed opportunities separate from confirmed defects. Do not claim a
static warning proves a runtime root cause without runtime evidence.

## Plans

For selected findings, write self-contained plans with exact paths, evidence,
current behavior, target behavior, ordered steps, scope boundaries, and
verification. If an analyzer publishes an authoritative fix recipe, use that
recipe instead of inventing one.

Keep the audit read-only until the user selects work. A concise table of
high-confidence findings is better than a padded list.
