---
name: actor-state-ownership
description: Design frontend state machines and actors with explicit ownership, cancellation, stale-result protection, and URL state boundaries.
---

# Actor state ownership

Use for XState or actor-based UI state, long-running requests, polling,
exports, uploads, subscriptions, retries, and browser storage.

## Choose the owner

- React owns rendering, event dispatch, and DOM-only refs or measurement.
- A machine owns durable state with legal transitions: phases, selection,
  queues, recoverable errors, and reset semantics.
- An actor owns external or concurrent behavior: requests, streams,
  cancellation, retries, timers, storage, and browser subscriptions.
- URL/search state owns shareable view state such as filters, sorting, page,
  view mode, and search text. Do not mirror it into local state with a
  write-back effect.
- Pure derived values belong in selectors, not duplicated context.

## Async protocol

- Give replaceable work an operation identity carried on every request,
  progress, completion, and failure event.
- Cancel the previous operation when replacement is intentional.
- Discard stale completions and chunks whose identity no longer matches the
  active operation.
- Make success, failure, cancellation, retry, and reset transitions explicit.
- Inject fetch, storage, clocks, ID generation, and browser subscriptions;
  reusable actors must not read framework globals directly.

## Composition

- Use one machine when transitions and invariants share a lifetime.
- Compose children when domains have separate lifetimes, failure modes, or
  update frequency.
- Route typed events through the parent; never copy child snapshots into parent
  context.
- Preserve protocol events at boundaries and expose narrow actor refs/selectors.

## Verification

Test legal transitions, cancellation, stale work, rejection, retry, parent
routing, child isolation, and concrete browser or storage adapters. Add browser
coverage only for rendered behavior and accessibility.

Reject actors for pure rendering or one-off DOM commands, application state in
`useState` when it has a lifecycle, and merged machines whose event and failure
semantics have not been proven equivalent.
