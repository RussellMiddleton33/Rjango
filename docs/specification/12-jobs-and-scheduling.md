# Jobs and scheduling

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part IV.

Durable jobs use at-least-once delivery, idempotency, observable retries, and transaction-aware dispatch. Queue backend choice remains open.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Durable dispatch and evolution

Transaction-coupled dispatch uses the [core outbox](23-durability-and-message-contracts.md); after-commit callbacks alone cannot survive process death reliably. Jobs/events use versioned envelopes. Consumers declare supported payload versions/upgrades and rolling-deployment windows. Unknown versions quarantine/dead-letter with diagnostics. Leases/retries/cancellation preserve at-least-once semantics; acknowledge after durable outcome. Delivery deduplication differs from business idempotency.


## Open decisions and interpretation

See the retained lock/open list. Backend defaults, scheduling leadership, payload evolution and exact APIs require validation and fuller design.



<!-- Source: iv section 23. -->
## Background Jobs

### Goal

Durable work should not depend on the lifetime of an HTTP request.

Examples:

```
send email
generate report
process upload
synchronize external system
rebuild search index
run AI task
create thumbnails
```

---

<!-- Source: iv section 24. -->
## Job Definition

Conceptually:

```
#[rjango::job]
async fn send_welcome_email(
    input: SendWelcomeEmail
) -> Result<()> {
    ...
}
```

Jobs must have structured input.

Avoid arbitrary closures as durable job payloads.

---

<!-- Source: iv section 25. -->
## Job Metadata

AMG representation:

```
job:users.send_welcome_email
```

contains:

```
input schema
retry policy
timeout
queue
concurrency policy
version
source
```

---

<!-- Source: iv section 26. -->
## Durable Queue Semantics

The framework should assume:

> Jobs may execute more than once.

We should **not** promise magical exactly-once execution.

Distributed systems make such guarantees extremely difficult once side effects are involved.

Default mental model:

```
at-least-once delivery
+
idempotent job design
```

---

<!-- Source: iv section 27. -->
## Idempotency

Rjango should make job idempotency straightforward.

Concept:

```
job key
deduplication key
idempotency record
```

Example:

```
send invoice #123
```

should be able to avoid sending the same invoice twice if a worker crashes after delivery but before acknowledgement.

---

<!-- Source: iv section 28. -->
## Dispatch

Conceptual API:

```
send_welcome_email::dispatch(
    &jobs,
    SendWelcomeEmail { user_id }
).await?;
```

Exact surface syntax remains OPEN.

---

<!-- Source: iv section 29. -->
## Transaction-Aware Dispatch

Critical feature:

```
Create User
   ↓
enqueue welcome email
   ↓
transaction rolls back
```

must not leave a job for a user that does not exist.

Rjango should support:

```
enqueue after transaction commit
```

as a first-class concept.

Potential architecture:

```
DB transaction
    │
    ├── application writes
    │
    └── transactional outbox entry
             │
          COMMIT
             │
             ▼
          queue
```

The transactional outbox pattern should be supported because it gives much stronger guarantees than naïvely writing to Redis during a database transaction.

---

<!-- Source: iv section 30. -->
## Job Backends

Core abstraction:

```
JobBackend
```

Potential providers:

```
PostgreSQL
Redis
external queue systems
```

Rjango should not couple the public job API to one queue.

Exact default backend remains OPEN until broader operational tradeoffs are specified.

---

<!-- Source: iv section 31. -->
## Job Workers

Conceptually:

```
rjango worker
```

Workers consume jobs separately from the HTTP server.

Application code should be shared.

---

<!-- Source: iv section 32. -->
## Worker Concurrency

Configurable:

```
global concurrency
per queue
per job type
per concurrency key
```

Example:

```
thumbnail:
20 concurrent

monthly_billing:
1 per account

external_vendor_sync:
maximum 5 total
```

---

<!-- Source: iv section 33. -->
## Job Timeouts

Jobs should have:

```
execution timeout
heartbeat/stall timeout
```

Long-running jobs may emit heartbeats.

Workers must distinguish:

```
slow but alive
```

from:

```
worker died
```

---

<!-- Source: iv section 34. -->
## Retries

Policy:

```
maximum attempts
initial delay
backoff
maximum delay
retryable error classes
non-retryable error classes
```

Example:

```
network timeout
→ retry

invalid user input
→ do not retry
```

---

<!-- Source: iv section 35. -->
## Backoff

Support:

```
fixed
linear
exponential
```

plus jitter to avoid thundering-herd retry storms.

---

<!-- Source: iv section 36. -->
## Dead-Letter State

Jobs that exceed retries move into a failed/dead state.

Operators can:

```
inspect
retry
discard
```

Actions are audited.

---

<!-- Source: iv section 37. -->
## Scheduled Jobs

Support one-time delayed jobs:

```
run at 2026-10-01T12:00
```

and recurring schedules.

---

<!-- Source: iv section 38. -->
## Cron-Style Scheduling

Schedule metadata:

```
timezone
expression
job
input
overlap behavior
misfire behavior
```

Timezones must be explicit.

Never silently assume server-local time for production schedules.

---

<!-- Source: iv section 39. -->
## Overlap Policy

Recurring job may specify:

```
Allow
Skip
Queue
Replace
Singleton
```

Example:

```
nightly import
```

should perhaps not start a second copy if yesterday's job is still running.

---

<!-- Source: iv section 40. -->
## Job Cancellation

Support cooperative cancellation where reasonable.

Cancellation semantics must be documented:

```
queued job
→ remove/cancel

running job
→ request cooperative cancellation
```

Cannot assume arbitrary external side effects can be rolled back.

---

<!-- Source: iv section 41. -->
## Job Progress

Optional structured progress:

```
0–100%
stage
message
metadata
```

Useful for:

```
exports
imports
media processing
AI workflows
```

---

<!-- Source: iv section 42. -->
## Job Observability

Trace:

```
request
   ↓
job dispatch
   ↓
queue
   ↓
worker
```

should preserve correlation IDs.

Job telemetry:

```
wait time
execution time
attempts
success/failure
queue depth
worker availability
```

---

<!-- Source: iv section 43. -->
## Job Testing

Test harness should support:

```
capture dispatched jobs
execute immediately
simulate retry
simulate failure
advance fake time
test schedules
test transaction commit behavior
```

Tests should not require real waiting.

---

<!-- Source: iv section 44. -->
## Django Mapping — Jobs

Django typically uses external tools such as Celery.

Rjango:

```
Django + Celery
→ Rjango Jobs + worker abstraction
```

Main difference:

> Job metadata, observability, permissions, testing and transaction-aware dispatch are framework concepts rather than an unrelated application bolted onto the side.

---

<!-- Source: iv section 132. -->
## Testing — Jobs

Required:

```
dispatch
transaction commit/rollback
worker crash
retry
backoff
dead-letter
idempotency
duplicate delivery
scheduling
timezone
concurrency
cancellation
graceful shutdown
queue disconnect/reconnect
```

Fault injection is especially important here.

---

<!-- Source: iv section 141. -->
## Current Lock Status — Jobs

#### LOCK

- durable jobs
- structured payloads
- worker processes
- at-least-once mental model
- idempotency support
- retries/backoff
- dead-letter state
- scheduling
- transaction-aware dispatch
- observability

#### OPEN

- default queue backend
- exact dispatch API
- storage representation
- outbox implementation details
