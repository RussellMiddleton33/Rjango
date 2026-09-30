# ADR 0003: PostgreSQL-first database design

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from the existing Rjango design conversation.

[ADR index](README.md) · [Canonical specification](../specification/03-models-and-orm.md) · [Status legend](../specification/README.md#status-legend)

## Context

A complete productive data layer needs a primary database target and must not hide useful capabilities behind premature portability.

## Decision

Design and validate PostgreSQL first. Preserve access to PostgreSQL features such as UUID, JSONB, arrays, numeric types, RETURNING and other database-specific capabilities. Future backend support must declare capabilities rather than silently promise equivalence.

## Consequences and boundaries

Real PostgreSQL integration and migration behavior define the initial evidence target. SQLite and MySQL are future possibilities, not current support claims.

## Alternatives and rationale

A lowest-common-denominator, every-database-first design was not selected. Detailed future backend scope remains open.

## Evidence still required

Constraint mapping, native types, schema introspection, migration safety and transaction semantics require real-database evidence.

This ADR records an architectural agreement, not completed implementation or a passing validation result. A future revision should link evidence or a superseding ADR rather than silently rewriting the decision history.
