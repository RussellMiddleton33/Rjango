# ADR 0005: Explicit database and transaction context

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from the existing Rjango design conversation.

[ADR index](README.md) · [Canonical specification](../specification/03-models-and-orm.md) · [Status legend](../specification/README.md#status-legend)

## Context

Ambient database discovery obscures dependencies and makes transactions, testing and future routing harder to reason about.

## Decision

Query entry points receive an explicit database/pool or transaction context, illustrated by `User::objects(&db)`. Use a cheap clonable database handle. Queries within a transaction must operate through that transaction context.

## Consequences and boundaries

Public syntax remains subject to ergonomic validation. Application and test code can see which context owns work, and future multi-database or tenant routing need not depend on task-local magic.

## Alternatives and rationale

Global or implicitly discovered context, including early `User::objects()` examples in the conversation, is superseded.

## Evidence still required

Transaction context reuse, savepoints, rollback after cancellation and pool reuse need real integration evidence.

This ADR records an architectural agreement, not completed implementation or a passing validation result. A future revision should link evidence or a superseding ADR rather than silently rewriting the decision history.
