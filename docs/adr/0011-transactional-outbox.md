# ADR 0011: Transactional outbox and transaction-owned durable effects

**Decision status:** SPEC-LOCKED.
**Evidence status:** VALIDATION REQUIRED.
**Recorded:** 2026-09-30, from Review Candidates 1 and 2.

[ADR index](README.md) · [Canonical specification](../specification/23-durability-and-message-contracts.md) · [Jobs](../specification/12-jobs-and-scheduling.md)

## Context

Jobs, domain events, email and webhooks triggered by database changes must not be lost when a process dies after commit, and must not fire when the transaction rolls back.

## Decision

Durable intents are written in the same database transaction as the business change, through methods on the transaction (`tx.emit`, `tx.dispatch`). Delivery is at least once, with stable message IDs and versioned envelopes. The 1.0 default job backend is a PostgreSQL queue table in the application database, where the job row is the outbox record and no relay runs. External brokers and providers are fed by a leased relay. Durable listeners are jobs keyed by (event, listener).

## Consequences and boundaries

There is no exactly-once promise and no cross-database atomicity. After-commit callbacks are only for explicitly non-durable work. Consumers need idempotency, and dedup expiry weakens duplicate protection.

## Alternatives and rationale

- **After-commit enqueue to Redis or a broker:** rejected, because a crash after commit loses the intent.
- **A mandatory relay even for a PostgreSQL queue:** rejected as needless complexity.

## Evidence still required

PostgreSQL queue throughput and `SKIP LOCKED` behaviour; relay crash windows; poison-message isolation; envelope upgrade determinism; retention.
