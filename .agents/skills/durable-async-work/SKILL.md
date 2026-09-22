---
name: durable-async-work
description: Decide when work needs a durable job contract and prevent unbounded, non-idempotent, or sequential async fan-out.
---

# Durable async work

Use when adding queues, workers, workflows, scheduled jobs, retries, polling,
batch operations, or deciding whether work should leave the request path.

## Inline or durable

Keep work inline when it is fast, its result is required immediately, and it
does not need independent retry or restart survival.

Make work durable when it is long-running, user-visible, restart-sensitive,
cancelable, retryable, a fan-out batch, or expensive enough to exceed the
request budget.

Do not wrap every provider call in a job. Durability is a user and operator
contract, not merely an implementation detail.

## Contract

A durable operation needs:

- an explicit lifecycle such as queued, running, succeeded, failed, or
  cancelled;
- an input schema and a distinct execution identity;
- idempotency or a claim mechanism before side effects;
- bounded progress and error information;
- explicit cancellation and retry semantics;
- a timeout, reaper, or reconciliation owner for abandoned work; and
- a clear separation between user-facing state and executor bookkeeping.

Create the user-facing record at the dispatch boundary before execution.
Retries create a new execution identity and retain the relationship to the
source attempt. Runtime-discovered child work belongs in an item-level model,
not in a misleading top-level job row.

## Batching and concurrency

Inspect every async loop and async collection callback. Classify it as:

- already batched;
- intentionally sequential, with a reason; or
- an N+1 or sequential fan-out that can use a list query, chunk, or explicit
  concurrency.

Do not issue one database or provider call per item when a bulk operation is
available. Do not rely on a library's sequential default for independent work;
state the concurrency policy in code.

## Verification

Test duplicate dispatch, cancellation, retry identity, stale completion,
timeout/reaping, partial failure, representative batch size, and the real
provider boundary. Keep intentionally sequential loops justified at the
exception site.
