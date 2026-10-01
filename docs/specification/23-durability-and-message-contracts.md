# Core durability and message contracts

[Master specification](README.md) · [Operations](22-application-operations-and-services.md)

**Status:** REVIEW CANDIDATE 1; architectural requirements adopted, implementation/backend guarantees VALIDATION REQUIRED.

## Transactional outbox

Any durable job/event/webhook/email intent atomically coupled to a database change can use the core transactional outbox. Persist business rows plus intent in the same database transaction. Rollback persists neither; commit persists both. An after-commit callback may wake a relay but is never the durable record. An intent committed in database A cannot make effects in database B atomic.

Relay workers claim committed records with bounded batches, lease/ownership semantics and retry policy. Publish then mark delivered; a crash after publication and before acknowledgement creates duplicates. Therefore delivery is at least once, never an unconditional exactly-once claim. Consumers use stable message IDs and domain idempotency; where possible, dedup record and consumer database mutation share one transaction. External providers need explicit idempotency or reconciliation. Dedup retention must cover replay/retry windows; expiry weakens duplicate protection.

Expose pending/delivered/retrying/quarantined states, oldest age, attempt counts and sanitized errors. Poison messages must not block unrelated messages indefinitely. Ordering is explicitly scoped per aggregate/partition only when implemented; global order is not promised. Retention/cleanup cannot erase pending intent; dead-letter replay preserves original identity/version and current authorization checks. Bounded queues do not silently discard durable intent. Broker and database outage recovery require real fault injection.

## Durable envelope

Jobs and durable events carry message_id, stable type, schema_version, created_at, correlation_id, causation_id and payload. Optional tenant, originating actor/subject/delegation references, operation ID and trace context are scoped/redacted; do not persist raw access tokens. Validate envelope and payload size before processing. Delivery attempt/lease state is operational metadata, not a new message identity.

Consumers declare supported versions and deterministic upgrade paths. Producers emit only versions supported during a declared rolling-deployment window. Unknown/invalid versions quarantine rather than deserialize into the current type by guesswork. Rollback and dead-letter replay must account for older binaries and long-retained payloads. Version changes include semantic changes, not only field shape. Choose retention/support windows explicitly; no duration has yet been validated.

Realtime has independently versioned protocol/schema negotiation because connected clients outlive deployments. It may reuse correlation/type conventions but remains ephemeral unless a persisted replay contract is explicitly declared. Queue durability does not imply recipient delivery or continuing authorization.

## Open evidence

Validate relay leasing, duplicate windows, ordering, consumer transactions, payload upgrade determinism, retained-message compatibility, storage limits and recovery. Exact table/API/backend defaults remain open. The outbox is a core framework primitive; adopting it does not require a universal broker or a distributed ACID promise.
