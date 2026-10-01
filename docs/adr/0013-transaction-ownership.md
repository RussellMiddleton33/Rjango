# ADR 0013: Transactions own their connection; commit consumes

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from Review Candidate 2.

[ADR index](README.md) · [ORM](../specification/03-models-and-orm.md) · Refines [ADR 0005](0005-explicit-database-context.md)

## Context

The transaction API had been treated as syntax, but its shape decides correctness:

- use of the outer handle inside a transaction;
- implicit versus explicit commit;
- unknown commit outcomes;
- concurrent queries on one connection;
- lifetime and error-inference failures in closures.

Django's ambient `atomic()` makes some of these mistakes likely for the primary audience.

## Decision

- **Ownership.** `db.begin()` returns a `Transaction` that owns one connection. Queries borrow it exclusively (`&mut`).
- **Commit and rollback.** `commit(self)` consumes the transaction and returns a typed outcome distinguishing definite failure from an unknown outcome. Drop without commit rolls back. Savepoints are nested guards with the same semantics.
- **Convenience.** `db.atomic(async |tx| ...)` is built on the guard with a fixed `rjango::Error` type. It requires async closures, so the MSRV is at least 1.85 with the Rust 2024 edition.
- **Diagnostic.** Using a pool handle while the same task holds an open transaction produces `RJG-DB-TX-OUTSIDE`: a warning in development and an error in tests.

## Consequences and boundaries

Concurrent queries and stream-plus-query on one transaction are compile errors. Use after commit is a compile error. These consequences teach moves and exclusive borrows. Unknown commit outcomes flow into operation reconciliation.

## Alternatives and rationale

- **Closure-only API:** rejected, because it hid commit outcomes and caused lifetime and inference errors.
- **Shared `&Transaction` with an internal mutex:** rejected, because of hidden serialization and deadlock risk.
- **Ambient task-local transaction:** rejected, because it conflicts with explicit context.

## Evidence still required

Ergonomics in handlers; compile-error quality for the ten most common mistakes; detection accuracy and cost of `RJG-DB-TX-OUTSIDE`; SeaORM/SQLx shared-transaction interoperability.
