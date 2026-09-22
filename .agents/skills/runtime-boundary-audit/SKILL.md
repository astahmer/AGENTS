---
name: runtime-boundary-audit
description: Audit ownership and dependency flow across UI, actors, services, adapters, external providers, and package boundaries.
---

# Runtime boundary audit

Use when reviewing service extraction, frontend state ownership, dependency
injection, adapter placement, provider lifecycle, or cross-package imports.

## First pass

- Inspect the current diff, nearby revisions, package exports, runtime entry
  points, and existing boundary checks before editing.
- Map each value and capability across its runtime boundary: user input,
  transport, persistence, domain, UI, browser, worker, or external provider.
- Identify the owner, lifetime, failure mode, and test boundary for every piece
  of state that the change moves.

## Ownership rules

- Keep React responsible for rendering, event wiring, and imperative DOM refs.
- Keep durable domain, navigation, persistence, subscription, and lifecycle
  state in the domain state machine or actor that owns its transitions.
- Keep I/O, browser globals, clocks, storage, network clients, and environment
  configuration behind injected adapters or composition roots.
- Provide service dependencies at the outer adapter or composition root. Do
  not pass service objects through ordinary business-function arguments.
- Keep orchestrators neutral. Concrete provider, framework, and ecosystem
  policy belongs in the owning adapter.
- Consume packages through declared public exports. Do not reach into another
  package's `src` tree or internal generated implementation from an app,
  script, test, or example.
- Decode external data once at the boundary into named validated types. Keep
  raw transport, persistence, domain, and public response types distinct.
- Enumerate fields crossing a boundary. Do not spread database rows or
  caller-controlled transport objects into public output.

## Implementation loop

1. Name the owner, lifetime, events, injected capabilities, and failure
   behavior before moving code.
2. Move the smallest complete state domain and delete the duplicate owner.
3. Preserve the public facade while keeping implementation details behind the
   intended package or adapter boundary.
4. Add a real provider, actor, or boundary test before broadening the change.
5. Verify the runtime path, not only TypeScript assignability.

## Reject a change when

- the same durable state is mirrored in multiple owners;
- service dependencies are smuggled through data arguments;
- a browser or provider capability leaks into reusable core code;
- an adapter decision appears in shared orchestration branches;
- an internal package path crosses a public boundary; or
- a new provider or actor has no behavioral lifecycle coverage.
