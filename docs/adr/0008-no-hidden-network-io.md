# ADR 0008: No hidden network I/O

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from the existing Rjango design conversation.

[ADR index](README.md) · [Canonical specification](../specification/20-cross-system-invariants.md) · [Status legend](../specification/README.md#status-legend)

## Context

Unexpected database, cache, object-store or external-service access behind property reads hides latency, failure and cost from humans and agents.

## Decision

Keep network I/O explicit in async operations. Relationship access cannot silently query; unloaded relations must be represented and handled explicitly. Storage, cache and service access follow the same rule. Declare side effects and durability at API boundaries.

## Consequences and boundaries

Typed relationship loading and N+1 diagnostics support deliberate data access. Domain events, durable jobs and ephemeral messages must not be conflated. Helper APIs may be ergonomic without hiding asynchronous effects.

## Alternatives and rationale

Django-style implicit relation fetching and hidden cross-system effects are not adopted. Exact unloaded-state types and helper names remain open.

## Evidence still required

Query-count, loaded-state, side-effect visibility and failure-path checks must show that the invariant holds across subsystems.

## Candidate 2 refinement

Loaded relations are `Loaded<M>` values with runtime-checked accessors ([ADR 0014](0014-loaded-relations.md)). Model hooks and schema validators are synchronous and receive no framework handle, so they cannot use framework I/O. Blocking foreign I/O inside them is not prevented by the compiler; it is a documented anti-pattern caught by the development stall detector. Cache keys and calls are explicit, awaited operations.

This ADR records an architectural agreement, not completed implementation or a passing validation result. A future revision should link evidence or a superseding ADR rather than silently rewriting the decision history.
