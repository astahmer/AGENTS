---
name: library-architecture
description: Keep public APIs small while making multi-implementation libraries extensible through explicit interfaces, registries, and substitution tests.
---

# Library architecture

Use when designing a reusable package, plugin system, adapter pipeline, or
public API.

## Public surface

- Treat the public API as a budget. Push back on additive surface that does
  not remove a real user problem or earn a written design decision.
- Prefer narrow public subpaths and cohesive domain objects over bags of
  exports, compatibility aliases, or generic utility buckets.
- Remove superseded internal contracts in the same logical change instead of
  keeping permanent deprecated projections “just in case.”
- Keep fast paths fast; do not force simple callers through the richest
  representation or lifecycle.

## Extension points

When a domain has more than one implementation, define a versioned interface
and registry/factory. A new implementation should register itself rather than
requiring every orchestrator branch to change.

An adapter contract should make identity, detection, configuration, parse,
emit, dependencies, capabilities, validation, and lossiness explicit where
those concepts apply. The orchestrator consumes neutral ports and metadata;
the adapter owns ecosystem-specific policy.

Discovery should be explicit and injectable. Avoid ambient global scans and
hardcoded vocabulary copies that happen to satisfy one fixture.

## Definition of done

- The policy has a named port and injectable default implementation.
- A composition root assembles it.
- A fake or minimal implementation can substitute without changing core
  orchestration.
- Tests prove registration, capability boundaries, and an unsupported case.
- Public exports and bundle impact are measured and documented.
