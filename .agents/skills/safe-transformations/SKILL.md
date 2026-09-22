---
name: safe-transformations
description: Run migrations, codemods, generators, and source-to-source transforms with previews, residuals, idempotence, and evidence.
---

# Safe transformations

Use when a command changes source files, generated output, schemas, migrations,
configuration, or a project from one representation to another.

## Before writing

- Pin the source revision and identify the input contract, configuration, and
  generated outputs.
- Inspect the route, adapter, target, and project package boundary.
- Prefer a dry run that emits an exact diff, report, and residual list.
- Preserve the source checkout; use a disposable output or isolated workspace
  when the operation is broad or difficult to reverse.

## Transformation rules

- Use registered adapters and neutral intermediate representations when more
  than one source or target exists. Do not grow bespoke source-to-target
  branches in the orchestrator.
- Keep ecosystem-specific parsing, vocabulary, and emission policy inside the
  owning adapter.
- Never silently drop an input, declaration, selector, theme dependency,
  plugin transform, or framework behavior. Preserve it, emit a structured
  residual, or stop with a clear unsupported case.
- Prefer verified metadata and official loaders over partial hardcoded copies
  of another tool's configuration.
- Make reruns idempotent. A second run over unchanged input should report zero
  changes or explain the remaining residuals.
- Review generated imports, paths, and target-native output as code, not merely
  as textually equivalent output.

## Proof of completion

Record the source revision, command, input/output counts, residuals, generated
artifacts, and focused validation. Check semantic behavior at the relevant
runtime boundary: database, server, browser, SSR, hydration, or streaming.

Success means the result builds and runs, expected residuals are explained,
generated metadata is fresh, and the transformation can be repeated from the
pinned source.
