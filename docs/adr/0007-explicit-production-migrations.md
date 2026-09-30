# ADR 0007: Explicit production migrations

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from the existing Rjango design conversation.

[ADR index](README.md) · [Canonical specification](../specification/04-migrations.md) · [Status legend](../specification/README.md#status-legend)

## Context

A model describes desired schema but cannot safely represent every historical or operational change to a live database.

## Decision

Keep desired AMG schema, historical migration schema and actual database schema distinct. Apply explicit source-controlled migrations in production, using dependency graphs, reviewable plans, risk classifications and independent authorization. Development schema sync must be opt-in and must not become production auto-sync.

## Consequences and boundaries

Renames require explicit intent; do not silently infer drop-and-add. Account for reversibility, data migration history, locks, drift, branch merges and operations that cannot run transactionally.

## Alternatives and rationale

Automatic production synchronization from current models and equating migration generation with application permission are rejected.

## Evidence still required

Deterministic diffs, historical replay, drift, reversibility limitations, transaction behavior and destructive-operation safety require PostgreSQL validation.

This ADR records an architectural agreement, not completed implementation or a passing validation result. A future revision should link evidence or a superseding ADR rather than silently rewriting the decision history.
