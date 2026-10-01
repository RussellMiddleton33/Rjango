# Application Operations / Services

[Master specification](README.md) · [Architecture review](../reviews/architecture-review-candidate-1.md)

**Status:** REVIEW CANDIDATE 1. Architectural requirements adopted; public syntax, trait design and backend behavior remain VALIDATION REQUIRED. This is a specification, not implementation planning.

## Purpose and boundary

An application operation is a reusable business use case with typed inputs/results, policy, transaction/durability semantics, audit and telemetry. Services group/cooperate on operations; domain rules remain ordinary testable Rust logic. Query and Command are semantic classifications, not a requirement to adopt CQRS or event sourcing. Queries declare scoped reads; commands declare mutations and durable effects. Incidental telemetry does not turn a query into a domain command.

HTTP, Admin, Realtime, CLI, Jobs and MCP are adapters to the same operation. They authenticate their caller, validate transport shape, enforce exposure/capability limits, construct a trusted context, invoke business logic and map its result into a transport schema. Registration in one adapter never exposes another adapter automatically. Internal helper functions need not all become registered operations.

## Context and descriptor

OperationContext carries actor, subject/delegation chain, trusted tenant, effective capabilities, explicit database/dependency handles, deadline/cancellation, correlation/causation IDs and invocation/idempotency identity. Do not accept caller-supplied context claims as authority. A worker adds its service identity without replacing origin. Transport information may aid audit but must not create privileged business bypasses.

AMG operation descriptors contain stable ID/version, input/output schema IDs, Query/Command class, policy and query scope, database/transaction boundary, effects with provenance/completeness, cancellation class, retry/idempotency contract, error descriptors and exposure classification. Edges identify which adapters invoke an operation and what it reads/writes/emits/requires. Raw SQL or opaque helpers declare incomplete effect knowledge. Descriptions are not executable authorization policies.

## Invocation contract

1. Admit within resource budgets; authenticate and validate input shape. Resolve trusted actor/subject/tenant and attenuated delegation.
2. Check adapter capability and current operation policy. Apply query scoping before reads/counts/pagination; validate object and related IDs within tenant boundaries.
3. Check relevant fingerprints and concurrency preconditions. Claim idempotency for eligible commands; identical retries reuse a durable outcome only after current access checks.
4. Validate domain rules. Open an explicit one-database transaction when required, with appropriate locking/version checks to avoid check/use races.
5. Persist mutation and outbox intent in that transaction. External email/webhook/broker calls are not inside its ACID guarantee.
6. Commit explicitly; record outcome/idempotency consistently. Return an explicit result projection. If commit outcome is unknown, return a diagnosable recoverable state and reconcile before retry.
7. Relay committed intents asynchronously, link telemetry and record audit. A committed operation cannot be undone merely because its response was lost.

Nested operations use an explicitly shared transaction or documented independent boundary. Do not open hidden transactions, retries or authorization escalation. Retrying a whole operation requires declared safety; non-idempotent external effects cannot be blindly repeated. Long workflows use durable state/compensation, not unbounded open database transactions. Audit persistence policy declares whether required audit failure blocks a command; successful durable mutations must not lose mandatory audit intent.

## Idempotency and concurrency

Scope keys to operation/version, environment, tenant and effective identity; store a canonical input digest and relevant preconditions. Same key with different input is a conflict. Concurrent identical attempts elect one executor; others get its outcome or bounded in-progress response. Persist pending/completed/failed/unknown states and declared retention. Database-backed outcomes should commit with database effects where feasible. Dedup expiry means indefinite exactly-once behavior is not promised. Recheck authorization before returning stored sensitive results.

## Adapter contracts

| Entry surface | Adapter responsibility |
| --- | --- |
| HTTP | Extract credentials/body, negotiate response, map typed output and RFC 9457 errors. |
| Admin | Explicit ReadOnly/DirectCRUD/OperationBacked mode; invariant-sensitive actions invoke commands. |
| Realtime | Negotiate protocol, validate message, revalidate subscription/command authority. |
| CLI | Resolve explicit environment and user/service authority; human and JSON output derive from one result. |
| Jobs | Decode supported envelope version, recover lease, resolve execution/origin policy, invoke then acknowledge durable outcome. |
| MCP | Validate local/remote trust and per-tool capability; enforce fingerprint/idempotency and sanitize output. |

## Example domain flow

CreateVenue accepts name and organization identity in an explicit input schema. Each exposed adapter invokes the same command. Trusted tenant scope checks organization ownership; CanCreateVenue policy and domain uniqueness apply. One database transaction inserts the venue, durable audit intent and VenueCreated outbox event. The result is a venue ID/projection; later consumers index it or send notifications idempotently. Realtime publishing is a delivery projection of the event and rechecks recipient permission. A lost HTTP/MCP response does not cause an unkeyed duplicate create. Exact Rust macro/trait spelling is open.

## Validation-required questions

What context ownership/lifetimes work across async operations and shared transactions? How do operation descriptors coexist with ordinary functions without excessive boilerplate? Which policy checks can compile to scopes? How are transaction-bound idempotency and mandatory audit persisted? What exception allows deliberate DirectCRUD? Validate adapter parity, denied/partial/unknown outcomes, concurrent replay, nested transaction behavior and shutdown through every surface. These are evidence requirements, not prototype authorization.
