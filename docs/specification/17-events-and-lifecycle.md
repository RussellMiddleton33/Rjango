# Events and lifecycle

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part IV.

Domain facts, durable jobs, ephemeral realtime messages, and framework lifecycle events have distinct delivery and failure contracts. Side effects and listener relationships remain inspectable.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Open decisions and interpretation

See the retained lock/open list. Listener execution/failure defaults, cycle safeguards, and exact lifecycle signatures remain open.



<!-- Source: iv section 96. -->
## Events

### Goal

Rjango needs an explicit event system without recreating the hidden complexity of unrestricted Django signals.

---

<!-- Source: iv section 97. -->
## Event Categories

Separate:

```
Lifecycle Event
Domain Event
Infrastructure Event
Audit Event
Realtime Event
```

These concepts serve different purposes.

---

<!-- Source: iv section 98. -->
## Domain Events

Represent something meaningful that occurred:

```
UserRegistered
VenuePublished
OrderPaid
```

Domain event is a data structure.

Example:

```
pub struct UserRegistered {
    pub user_id: Uuid,
}
```

---

<!-- Source: iv section 99. -->
## Event Dispatch Semantics

We should support both:

```
in-process event
durable event
```

but they must have visibly different APIs/metadata.

Developers must know whether an event survives process failure.

---

<!-- Source: iv section 100. -->
## Transactional Domain Events

Important business events often need transaction consistency.

Example:

```
Order marked Paid
+
OrderPaid event
```

should not produce an event if DB commit fails.

Again, transactional outbox support is the stronger architecture.

---

<!-- Source: iv section 101. -->
## Event Listeners

Listeners explicitly register.

Conceptually:

```
UserRegistered
  ├── SendWelcomeEmail
  ├── CreateAuditRecord
  └── StartOnboarding
```

AMG represents these edges.

---

<!-- Source: iv section 102. -->
## Hidden Side Effects

`save()` should not unpredictably trigger dozens of global listeners.

Rjango should discourage:

```
model saved
→ arbitrary invisible application behavior
```

for core business logic.

Domain services/events make causality clearer.

---

<!-- Source: iv section 103. -->
## Model Lifecycle Hooks

Local hooks can exist:

```
before_insert
after_insert
before_update
after_update
before_delete
after_delete
```

Their intended use should be narrow:

```
normalization
local invariants
small model-specific behavior
```

Not major orchestration.

---

<!-- Source: iv section 104. -->
## Event Cycles

Framework should detect or help diagnose:

```
A emits B
B emits A
```

when possible.

Runtime should protect against obvious infinite synchronous dispatch recursion.

---

<!-- Source: iv section 105. -->
## Event Failure Policy

Synchronous listeners:

```
listener failure
→ operation may fail
```

Durable asynchronous listeners:

```
event committed
→ listener retries independently
```

This distinction must be explicit.

---

<!-- Source: iv section 106. -->
## Application Lifecycle

Framework lifecycle:

```
Build
 ↓
Register
 ↓
Resolve
 ↓
Validate
 ↓
Initialize
 ↓
Ready
 ↓
Serve
 ↓
Drain
 ↓
Shutdown
```

Each phase has defined guarantees.

---

<!-- Source: iv section 107. -->
## Startup Hooks

Useful for:

```
initialize SDK
warm cache
verify dependency
register external resource
```

But database schema mutation should **not** occur silently in normal startup hooks.

---

<!-- Source: iv section 108. -->
## Graceful Shutdown

On shutdown:

```
stop accepting new traffic
        ↓
drain HTTP requests
        ↓
stop taking new jobs
        ↓
allow running work to finish up to timeout
        ↓
close realtime connections
        ↓
flush telemetry
        ↓
close pools
```

---

<!-- Source: iv section 137. -->
## Testing — Events

Required:

```
ordering
listener failure
transaction rollback
durable delivery
cycles
duplicate processing
listener registration
shutdown
```

---

<!-- Source: iv section 146. -->
## Current Lock Status — Events

#### LOCK

- domain events
- explicit listeners
- durable vs in-process distinction
- transactional compatibility
- limited model hooks
- lifecycle events

#### OPEN

- exact event bus API
- event persistence format
- cross-service event conventions
