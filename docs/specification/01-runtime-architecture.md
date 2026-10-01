# Runtime architecture

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Initial master specification.

Tokio supplies the multi-threaded async runtime. Rjango supplies approachable APIs, safe shared state, explicit resource ownership, blocking-work isolation, and lifecycle coordination.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Cancellation and shutdown

Important operations declare CancellationSafe, CancellationLeavesRecoverableState, or NonInterruptiblePhase semantics. Cancelling a future stops waiting, not necessarily server-side work. Cancellation before commit cannot be reported as success; cancellation during commit can leave an unknown outcome that must be reconciled through durable operation/idempotency state before retrying. Pool recovery and blocking-task termination cannot be assumed.

Shutdown marks readiness false, stops ingress and worker intake, drains tracked requests/jobs, signals cancellation, records recoverable work, flushes telemetry and closes dependencies. Every phase has a finite configurable deadline inside one total budget. Numeric defaults remain PROPOSED. Noninterruptible phases need bounded timeouts or explicit recovery. Unfinished leased jobs recover after expiry; committed outbox intents survive; request-spawned tasks are never durable work.

### Unified bounded-resource/backpressure model

HTTP bodies/connections, database pools, blocking executors, workers, outbox relays, realtime subscribers, MCP requests and telemetry exporters declare concurrency, queue-count/byte, payload and time limits, including tenant/identity quotas where needed. Admit before expensive allocation. Saturation selects bounded wait, rejection/retry-after, ephemeral drop or disconnect explicitly. Durable intent is never silently dropped: storage exhaustion rejects admission or fails its transaction. Avoid waiting on nested limits while holding scarce database connections. Expose saturation, queue age and rejection metrics with bounded cardinality. Global quotas and numeric defaults require validation.


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (H2, M11). Numeric values are PROPOSED; claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment where they conflict.

### Who assigns cancellation classes, and the defaults

The operation descriptor carries the cancellation class ([22](22-application-operations-and-services.md)); macros assign defaults and developers override at Tier 2.

- **Query operations** are CancellationSafe: when the caller disconnects or the deadline passes, the future is dropped.
- **Command operations** are shielded from the connection. The adapter runs the command in a framework-tracked task that continues if the client disconnects, until it completes or reaches its operation deadline. Its result is recorded (and stored for idempotent commands) even if nobody receives it.
  - Deadline expiry before commit rolls back and reports a timeout.
  - Expiry during commit yields `CommitOutcomeUnknown`.
  - Shielded tasks are counted in the shutdown drain budget, have a per-instance concurrency bound, and never outlive the process. They are not durable work; durable continuation uses the outbox and jobs.
- **NonInterruptiblePhase** regions (commit, migration DDL steps) are bounded by their own timeouts and recovered through the ledger or idempotency state.

This replaces the HTTP-section implication ([06](06-http-and-routing.md)) that every handler future is cancelled when the client disconnects. Django's synchronous views ran to completion; Rjango keeps that expectation for writes and drops it for reads.

### Blocking work and password hashing

Memory-hard password hashing, template rendering of large documents, image work and synchronous libraries run on the bounded blocking executor. Admission is bounded, so a login flood queues or rejects instead of starving runtime threads ([09](09-authentication.md)). Development mode reports runtime-thread stalls above a configurable threshold (PROPOSED 50 ms) with the span that blocked.

### Application state

Framework resources (pools, clients, configuration, AMG) are shared through owned, cheaply clonable handles. Application state is registered with `.state(T)` (`T: Send + Sync + 'static`) and extracted with `State<T>`. Mutable shared state uses explicit `Arc<Mutex<_>>`/`RwLock`, atomics, or framework primitives (rate-limit counters, request-local caches), taught as a Rust concept ([24](24-developer-experience-and-diagnostics.md)). The principle "without forcing developers to deal with `Arc` or locks" (in [00](00-vision-and-principles.md)) applies to framework-managed resources only.

### Validation required

Disconnect behaviour of Hyper/Axum on HTTP/1.1 and HTTP/2; the cost and bounds of shielded commands; stall detector overhead; blocking-pool sizing under login floods.

## Open decisions and interpretation

Public blocking-helper syntax, detailed cancellation budgets, runtime tuning, and the full operational/performance contract require further design or validation.

## Runtime contract from the design discussion

- Use Tokio's multi-thread runtime and lightweight async tasks; a request does not own an operating-system thread.
- Share application resources through cheap handles and `Arc` where appropriate. State must satisfy the relevant `Send` and `Sync` constraints; connection pools are shared, not recreated per request.
- Keep database and network waits asynchronous. Isolate synchronous libraries and expensive CPU work in bounded blocking execution or explicit jobs. The exact blocking-helper API remains proposed.
- Expose normal application concepts to users; low-level task scheduling, futures, locks, and runtime tuning should not dominate common handlers.
- Distinguish concurrency (progress while other work waits) from parallelism (work on multiple cores). Neither implies a throughput guarantee without measurement.
- Integrate web and worker processes with the same application model, graceful shutdown, timeouts, and cancellation policies.

Django is not inherently single-threaded: deployment models can use threads, processes, and async execution. The Rjango decision is to make async, concurrency, and safe shared state native architectural assumptions.

See [Tokio ADR](../adr/0001-tokio-runtime.md), [HTTP](06-http-and-routing.md), [jobs](12-jobs-and-scheduling.md), and [lifecycle](17-events-and-lifecycle.md).

<!-- Source: master section 4. -->
## Runtime Architecture

### Request model

Each incoming request becomes an asynchronous task.

```
Request A ─┐
Request B ─┼── Tokio scheduler ── CPU cores
Request C ─┤
Request D ─┘
```

While Request A waits for PostgreSQL:

```
A → waiting

CPU executes B/C/D

database returns

A → resumes
```

This allows extremely high I/O concurrency without requiring one OS thread per connection.

---

<!-- Source: master section 5. -->
## Blocking Work

Blocking or CPU-heavy operations must not accidentally stall Tokio runtime threads.

Rjango should expose:

```
rjango::task::blocking(|| {
    perform_cpu_heavy_operation()
}).await?;
```

Potential future macro:

```
#[rjango::blocking]
fn create_thumbnail(...) {
}
```

Documentation must clearly distinguish:

#### Async workloads

```
Database
Redis
HTTP APIs
S3
Network operations
Async filesystem APIs
```

#### Blocking/CPU workloads

```
Image processing
Compression
Large PDF generation
Video operations
CPU-heavy calculations
Synchronous legacy libraries
```

---

<!-- Source: master section 8. -->
## Application Boot

Desired:

```
use rjango::prelude::*;

#[rjango::main]
async fn main() -> Result<()> {
    Rjango::new()
        .app(users::app())
        .app(venues::app())
        .run()
        .await
}
```

Potential generated version may be even simpler:

```
#[rjango::main]
async fn main() {
    app().run().await;
}
```

Rjango should handle:

- configuration
- logging
- database initialization
- connection pools
- metadata registration
- routes
- graceful shutdown
- framework checks
- lifecycle hooks
