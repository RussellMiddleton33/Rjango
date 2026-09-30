# ADR 0004: SeaORM 2.x over SQLx 0.9 with direct escape hatch

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from the existing Rjango design conversation.

[ADR index](README.md) · [Canonical specification](../specification/03-models-and-orm.md) · [Status legend](../specification/README.md#status-legend)

## Context

Rjango needs Django-like model/query ergonomics, async database access and metadata ownership without reimplementing ORM and driver machinery.

## Decision

Select SeaORM 2.x as the ORM foundation over SQLx 0.9. Rjango owns public model/query syntax, metadata, compatibility, diagnostics, migration policy and instrumentation. Expose deliberate direct SQLx/raw SQL and optional compile-checked SQL escape hatches.

## Consequences and boundaries

Ordinary public APIs should not require SeaORM types. Exact compatible versions, pool/transaction sharing and adapter boundaries must be established before dependency pins or stability claims. The version pairing is a design target, not a verified dependency graph.

## Alternatives and rationale

Diesel was discussed as a legitimate alternative with strong query typing. The conversation preferred SeaORM’s fit with model-first, async, relationship and metadata goals; no comparative prototype results were supplied.

## Evidence still required

Macro/API ergonomics, loaded-state handling, raw-query interoperability, cancellation, N+1 behavior and performance remain VALIDATION REQUIRED. The former “PROVISIONALLY VALIDATED” label did not establish passing tests.

This ADR records an architectural agreement, not completed implementation or a passing validation result. A future revision should link evidence or a superseding ADR rather than silently rewriting the decision history.
