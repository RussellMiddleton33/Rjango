# Runtime architecture

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Initial master specification.

Tokio supplies the multi-threaded async runtime. Rjango supplies approachable APIs, safe shared state, explicit resource ownership, blocking-work isolation, and lifecycle coordination.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

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
