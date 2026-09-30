# Cross-system invariants

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III, Part IV.

Subsystems share metadata, identity, configuration, errors, observability, audit, and lifecycle policy. These rules constrain all feature specifications.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Open decisions and interpretation

A final cross-system design review remains outstanding. These invariants must be reconciled with each remaining area before implementation planning.



<!-- Source: iii section 114. -->
## Cross-System Invariants Established So Far

At this stage, we can establish several extremely important invariants.

### Invariant 1

```
Source code
→ AMG
→ projections
```

Never:

```
Admin/OpenAPI/MCP
→ independent competing schema
```

---

### Invariant 2

Database models and API schemas are separate.

```
Database ≠ Public API
```

---

### Invariant 3

Authentication and authorization are separate.

```
Identity ≠ Permission
```

---

### Invariant 4

Metadata access and data access are separate.

```
Know User.email exists
≠
Read users' email addresses
```

---

### Invariant 5

Durable background work does not live inside request lifecycle assumptions.

```
HTTP request
≠
job queue
```

---

### Invariant 6

Production schema mutation requires explicit migration intent.

```
model change
≠
automatic production DDL
```

---

### Invariant 7

Rjango abstractions never eliminate lower-level escape hatches.

```
Rjango
 ↓
SeaORM / Axum / Tower / SQLx
```

remain accessible intentionally.

---

<!-- Source: iii section 115. -->
## Current Design Status

### FOUNDATION

Runtime architecture
**LOCKED AT SPEC LEVEL**

Application Metadata Graph
**LOCKED CORE / DETAILS OPEN**

ORM architecture
**SPEC-LOCKED · VALIDATION REQUIRED**

---

### MIGRATIONS

Explicit migration system
**LOCKED**

DAG architecture
**LOCKED**

Safety classification
**LOCKED**

Historical model semantics
**LOCKED**

Exact migration file representation
**OPEN**

---

### APPLICATIONS

Explicit registration
**LOCKED**

Reusable app model
**LOCKED**

Exact builder syntax
**OPEN**

---

### HTTP

Axum/Tower foundation
**LOCKED**

Async handlers
**LOCKED**

Typed extractors
**LOCKED**

Route AMG metadata
**LOCKED**

Exact macro spelling
**OPEN**

---

### SCHEMAS

Separate model/schema types
**LOCKED**

Three-state PATCH semantics
**LOCKED**

Validation metadata
**LOCKED**

Exact derive/attribute syntax
**OPEN**

---

### APIs

Explicit APIs + optional resource abstraction
**LOCKED**

OpenAPI generated from AMG
**LOCKED**

TypeScript SDK generation
**LOCKED**

Exact resource DSL
**OPEN**

---

### AUTH

Identity/authorization separation
**LOCKED**

Sessions
**LOCKED**

API keys
**LOCKED**

OAuth/OIDC architecture
**LOCKED**

Passkey-compatible design
**LOCKED**

Exact permission composition syntax
**OPEN**

---

### SECURITY

Secure production defaults
**LOCKED**

MCP capability isolation
**LOCKED**

Secret redaction
**LOCKED**

Trusted proxy model
**LOCKED**

Structured audit events
**LOCKED**

Unsafe-Rust-requires-justification policy
**LOCKED**

---

<!-- Source: iii section 117. -->
## Architectural Result So Far

Rjango is becoming:

```
                           Rjango
                              │
         ┌────────────────────┼─────────────────────┐
         │                    │                     │
       Human                 AI                  Runtime
      Developer             Agent                 App
         │                    │                     │
         └────────────┬───────┴────────────┬────────┘
                      │                    │
                     AMG              Security Policy
                      │                    │
        ┌─────────────┼────────────────────┼────────────┐
        │             │                    │            │
      Models      HTTP/API              Auth         Jobs
        │             │                    │
        ▼             ▼                    ▼
     SeaORM          Axum              Identity
        │             │
       SQLx          Tower
        │             │
        └──────────┬──┘
                   ▼
                Tokio
                   │
             PostgreSQL
```

The key idea continues to hold:

> **Rjango does not replace Rust's excellent low-level ecosystem. It gives those pieces a unified application model, conventions, safety policy, metadata graph, developer experience, documentation system, and AI interface.**

<!-- Source: iv section 1. -->
## Cross-System Principle

These systems must not become independent mini-frameworks.

They should share:

```
Application Metadata Graph
Identity / Authorization
Configuration
Observability
Errors
Audit Events
Lifecycle
Documentation
MCP Permissions
```

Conceptually:

```
                   Application Metadata Graph
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
        Admin               Jobs             Realtime
          │                  │                  │
        Auth              Events             Auth
          │                  │                  │
          └──────────┬───────┴───────┬─────────┘
                     │               │
                   Audit         Observability
                     │               │
                     └───────┬───────┘
                             │
                       Configuration
```

---

<!-- Source: iv section 148. -->
## New Cross-System Invariant: Durable Side Effects

Operations that must survive request/process failure should flow through durable infrastructure.

```
HTTP handler
    │
    ├── DB transaction
    │
    └── durable intent
            │
         COMMIT
            │
            ▼
          Job/Event
```

Not:

```
HTTP handler
  ├── save DB
  ├── send email
  ├── call webhook
  └── hope nothing crashes
```

---

<!-- Source: iv section 149. -->
## New Cross-System Invariant: Side-Effect Visibility

Rjango should make significant side effects structurally visible.

The AMG should increasingly answer:

```
What can this route mutate?

What jobs can it dispatch?

What events can this operation emit?

Which listeners respond?

Which external systems may be called?
```

Not all of this will be statically knowable, but framework-provided declarations should expose as much as possible.

---

<!-- Source: iv section 150. -->
## New Cross-System Invariant: Explicit Durability

Every async-looking operation must be clear about whether it is:

```
in-memory
process-local
distributed
durable
```

Examples:

```
tokio::spawn
→ process-local, not durable

realtime broadcast
→ usually ephemeral

Rjango Job
→ durable

domain event via durable outbox
→ durable
```

This distinction belongs prominently in both human and AI documentation.

---

<!-- Source: iv section 151. -->
## New Cross-System Invariant: Auth Everywhere

HTTP authorization rules must not be bypassed by alternative interfaces.

Security model applies independently to:

```
HTTP
Admin
WebSocket
MCP
Jobs where relevant
CLI privileged actions
```

Each interface identifies an actor/capability context and checks appropriate policy.

---

<!-- Source: iv section 152. -->
## New Cross-System Invariant: Observable by Default

Framework-managed operations should create structured telemetry without application developers manually instrumenting basic framework actions.

Examples:

```
HTTP request
DB query
cache lookup
job execution
storage upload
email send
realtime connection
MCP tool call
```

All should be traceable.

---

<!-- Source: iv section 153. -->
## New Cross-System Invariant: No Hidden Network I/O

Ordinary property access should never unexpectedly trigger:

```
database query
cache request
object-storage request
HTTP call
email
queue operation
```

Network/durable operations should involve explicit async calls.

This is a major Rust-friendly improvement over some dynamic framework patterns.

---

<!-- Source: iv section 154. -->
## Rjango's Emerging Programming Model

A typical request might eventually look conceptually like:

```
#[rjango::post("/venues")]
#[permission(CanCreateVenue)]
async fn create_venue(
    db: Db,
    jobs: Jobs,
    identity: Identity,
    Json(input): Json<CreateVenue>,
) -> Result<VenueResponse> {

    let venue = db.transaction(|tx| async move {
        let venue = Venue::objects(tx)
            .create(input.into())
            .await?;

        VenueCreated::publish_after_commit(
            tx,
            VenueCreated { venue_id: venue.id }
        ).await?;

        Ok(venue)
    }).await?;

    Ok(venue.into())
}
```

And elsewhere:

```
#[rjango::listener]
async fn on_venue_created(
    event: VenueCreated,
    jobs: Jobs,
) -> Result<()> {
    rebuild_search_index::dispatch(
        &jobs,
        RebuildVenue { venue_id: event.venue_id }
    ).await?;

    Ok(())
}
```

The exact syntax is not locked.

The architectural behavior is:

```
explicit dependencies
explicit async I/O
transaction-safe events
durable jobs
structured metadata
observable execution
typed payloads
```

---

<!-- Source: iv section 155. -->
## Architectural North Star

Rjango should eventually let a human or AI inspect a complex feature and see something like:

```
POST /venues
│
├── accepts
│   └── CreateVenue
│
├── requires
│   └── CanCreateVenue
│
├── writes
│   └── venues.Venue
│
├── emits
│   └── VenueCreated
│
└── VenueCreated listeners
    │
    ├── dispatch RebuildVenueIndex
    ├── create audit event
    └── broadcast realtime update
```

That degree of structural transparency would be extraordinarily valuable for:

- onboarding
- debugging
- security review
- documentation
- code review
- impact analysis
- AI agents

and it continues the fundamental Rjango idea:

> **A modern framework should understand the application it is running.**
