# Cross-system invariants

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III, Part IV.

Subsystems share metadata, identity, configuration, errors, observability, audit, and lifecycle policy. These rules constrain all feature specifications.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Candidate 1 shared invariants

All six surfaces share operation rules; transport exposure grants no business authority. AMG exposes provenance/completeness/projection identity. Rows, descriptors and loaded values are distinct. Stable migration IR/checksums/single-migrator/step recovery are required. Database plus durable intent commit in one database; delivery is at least once. Identity/delegation, tenant/query scopes, wire types, cancellation and resource bounds span every subsystem. Core MCP never returns raw secrets; production has zero-data-access defaults. See [review disposition](../reviews/architecture-review-candidate-1.md).


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (C1, C2, H1, H9, M10). Syntax is illustrative; claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment and retained sketches below.

### Candidate 2 shared invariants

1. **Every entry point is an operation.** Plain handlers are implicit operations with documented defaults ([22](22-application-operations-and-services.md)).
2. **Deny by default.** Every exposed surface declares a policy or `public` ([10](10-authorization-and-security.md)).
3. **Scoped by default.** The default database handle applies tenant and policy scopes; unscoped access is a declared, audited capability ([03](03-models-and-orm.md)).
4. **Transactions own their connection.** Access is exclusive, commit consumes the transaction, drop rolls back, and an unknown outcome is a typed error.
5. **Durable effects go through the transaction.** `tx.emit` and `tx.dispatch`; never after-commit callbacks for durable work ([23](23-durability-and-message-contracts.md)).
6. **Loaded relations are explicit values.** `Loaded<M>` with runtime-checked accessors; rows never contain relation fields.
7. **Commands run to completion or deadline.** Queries cancel on disconnect ([01](01-runtime-architecture.md)).
8. **Diagnostics are API.** Compile-time versus startup checks are classified honestly ([24](24-developer-experience-and-diagnostics.md)).
9. **Production is the fail-safe environment.** Development settings never flow into it ([18](18-configuration.md)).
10. **Schema changes are rolling-deploy aware.** Expand/contract tags and readiness gating ([04](04-migrations.md)).

### Conforming programming model

The "Rjango's Emerging Programming Model" example below is SUPERSEDED. It used no operation, an unscoped create, after-commit naming, non-transactional job dispatch and unused extractors. The conforming shapes are below (illustrative syntax).

**Tier 0: six concepts** (model, schema, handler, policy, `Result`/`?`, `.await`):

```
#[rjango::model(tenant = organization_id)]
pub struct Venue {
    #[primary_key]
    #[default = uuid_v7]
    pub id: Uuid,
    pub organization_id: Uuid,
    #[unique]
    pub slug: String,
    #[max_length = 200]
    pub name: String,
    #[version]
    pub version: i64,
}

#[rjango::schema(from = Venue, input, fields(slug, name))]
pub struct CreateVenue;

#[rjango::schema(from = Venue, output, fields(id, slug, name))]
pub struct VenueResponse;

#[rjango::policy]                                    // context-only policy: no object exists yet
async fn can_create_venue(ctx: &Ctx) -> Decision { /* e.g. member role in ctx's tenant */ }

#[rjango::post("/venues", policy = can_create_venue)]
async fn create_venue(ctx: Ctx, Json(input): Json<CreateVenue>) -> rjango::Result<VenueResponse> {
    let venue = Venue::objects(&ctx.db()).create(input.into()).await?;   // scoped; tenant key from ctx
    Ok(VenueResponse::from(&venue))
}
```

`NewVenue` omits the tenant key, the defaulted primary key and the `#[version]` field ([03](03-models-and-orm.md)), so `CreateVenue` converts into it. The scoped handle fills the tenant from `Ctx`, so a client cannot choose a tenant. The handler is an implicit Command operation: its policy is required, it is shielded from disconnect, it reports Problem Details errors, and its effect (writes `venues.Venue`) is generated into the AMG. A single `create` runs in its own implicit statement transaction.

**Tier 1: adds a transaction, a durable event and a listener:**

```
#[rjango::post("/venues", policy = can_create_venue)]
async fn create_venue(ctx: Ctx, Json(input): Json<CreateVenue>) -> rjango::Result<VenueResponse> {
    let mut tx = ctx.db().begin().await?;                        // scoped to ctx's tenant
    let venue = Venue::objects(&mut tx).create(input.into()).await?;
    tx.emit(VenueCreated { venue_id: venue.id }).await?;         // outbox intent, same commit
    tx.commit().await?;                                          // consumes tx; unknown outcome is typed
    Ok(VenueResponse::from(&venue))
}

#[rjango::listener]                                              // durable: a job per (event, listener)
async fn reindex_venue(ctx: JobCtx, event: VenueCreated) -> rjango::Result<()> {
    search::reindex_venue(&ctx, event.venue_id).await
}
```

The transaction is explicit because the commit outcome is part of the contract. `db.atomic(async |tx| ...)` is the equivalent convenience form.

## Open decisions and interpretation

The first cross-system review is incorporated in Candidate 1. Independent review and reconciliation with remaining design areas remain outstanding; no implementation planning is authorized.



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

> **SUPERSEDED (Candidate 2):** see "Conforming programming model" in the amendment above. This sketch is retained for history only.

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
