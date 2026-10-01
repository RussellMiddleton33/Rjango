# API framework

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III.

The API layer supports explicit handlers and opt-in resource APIs, with allowlisted exposure, typed filtering, stable errors, and AMG-driven OpenAPI and clients.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Shared mutation contracts

APIs invoke [operations](22-application-operations-and-services.md), use [wire types](07-schemas-and-validation.md) and [Problem Details](06-http-and-routing.md). Idempotency belongs to the operation and survives adapter differences. Stale fingerprints fail preconditions; hashes never grant authority. Compatibility covers behavior/errors/types as well as routes.


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (H10, H6, M1, L2). Syntax is illustrative; claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment and retained sketches below.

### Resources are generated operations

A resource (the DRF `ViewSet` counterpart) generates up to five Tier 0 operations: list, retrieve, create, update, delete. Each is an ordinary [operation](22-application-operations-and-services.md) with:

- **Policy.** A policy per action, or one policy for all; required (deny by default).
- **Data access.** The scoped `Db`, so list/count/pagination are tenant- and policy-scoped.
- **Schemas.** Derived schemas ([07](07-schemas-and-validation.md)): `output`, `input`, `patch`.
- **Filtering and ordering.** Executed through the `DynamicModel` layer ([03](03-models-and-orm.md)) against declared `filter`/`ordering` allowlists only.
- **Concurrency.** Optimistic concurrency through `#[version]` and ETag/`If-Match` on update and delete when the model declares a version.
- **Pagination.** Page-number by default, cursor opt-in, with `Count` totals.
- **Overrides.** Any action can be replaced by a named operation or disabled.

Illustrative:

```
#[rjango::resource(
    model = Venue,
    output = VenueResponse,
    input = CreateVenue,
    patch = UpdateVenue,
    policy = venues::can_manage_venue,
    filter(active, name = icontains),
    ordering(name, created_at),
)]
pub struct VenueResource;
```

### Idempotency belongs to the operation

`#[idempotent]` with pluggable storage (below) is SUPERSEDED. Idempotency is declared on the operation, and its records live in the operation's database. A claim is written in the same transaction as the effects, so outcome and effects commit together. Other storage (such as Redis) is not supported for idempotency, because it cannot commit atomically with PostgreSQL.

### Errors and deprecation

The "API Error Contract" below is expressed as Problem Details types and codes ([24](24-developer-experience-and-diagnostics.md)). Deprecation uses `#[rjango::deprecated(since, remove, replacement)]`: the unqualified `#[deprecated(...)]` with `remove`/`replacement` keys collides with Rust's built-in attribute.

## Open decisions and interpretation

Idempotency storage, versioning defaults, final resource syntax, SDK scope, and precise compatibility rules remain open.



<!-- Source: iii section 59. -->
## API Framework

Rjango should support two layers:

### Explicit handlers

Maximum control:

```
#[get("/venues")]
async fn list_venues(...) { ... }
```

### Resource APIs

High productivity:

```
#[rjango::resource]
#[model(Venue)]
pub struct VenueResource;
```

These layers coexist.

---

<!-- Source: iii section 60. -->
## Resource APIs

Potential metadata configuration:

```
#[rjango::resource(
    model = Venue,
    input = CreateVenue,
    output = VenueResponse
)]
pub struct VenueResource;
```

Default generated capabilities may include:

```
list
retrieve
create
update
delete
```

All must be individually configurable.

---

<!-- Source: iii section 61. -->
## Explicit Exposure

CRUD should not automatically expose every model.

Models are private persistence structures unless explicitly surfaced.

This prevents:

```
create model
→ accidentally public API
```

---

<!-- Source: iii section 62. -->
## Filtering

Typed filter schemas:

```
#[rjango::filter(Venue)]
pub struct VenueFilter {
    pub active: Option<bool>,

    #[lookup = "icontains"]
    pub name: Option<String>,
}
```

Longer term generated query types may eliminate some boilerplate.

---

<!-- Source: iii section 63. -->
## Ordering

Whitelisted fields only.

Example:

```
?ordering=name,-created_at
```

The client should not be able to inject arbitrary SQL identifiers.

---

<!-- Source: iii section 64. -->
## Pagination

Support:

```
page-number
cursor
```

Cursor pagination should be encouraged for high-scale frequently changing datasets.

Pagination metadata is standardized.

---

<!-- Source: iii section 65. -->
## API Error Contract

One framework-wide envelope.

Errors have stable machine codes.

Example:

```
validation_error
not_found
not_authenticated
permission_denied
conflict
rate_limited
internal_error
```

Application-specific errors can extend the namespace.

---

<!-- Source: iii section 66. -->
## OpenAPI

OpenAPI is derived from AMG.

```
Routes
+
Schemas
+
Auth
+
Errors
+
Parameters
=
OpenAPI
```

The OpenAPI document is a **projection**, not another source of truth.

---

<!-- Source: iii section 67. -->
## Generated SDKs

TypeScript first.

Generated client should include:

```
types
requests
responses
errors
auth integration hooks
pagination
```

Generated code should be deterministic.

Fingerprint:

```
API graph fingerprint
```

allows no-op generation when nothing changed.

---

<!-- Source: iii section 68. -->
## Idempotency

> **SUPERSEDED (Candidate 2):** idempotency is declared on the operation and stored in the operation's database, not in pluggable storage.

Rjango should provide first-class idempotency for mutation APIs.

Example:

```
#[idempotent]
#[post("/payments")]
```

using:

```
Idempotency-Key
```

with pluggable backing storage.

Especially useful for:

- payment operations
- retries
- mobile networks
- agent-driven actions

---

<!-- Source: iii section 69. -->
## API Versioning

Rjango should support versioning but not prescribe one universal strategy.

Possible:

```
URL:
 /api/v1/

Header:
 Accept-Version

host/subdomain
```

A project chooses policy.

AMG associates routes/schemas with API version.

---

<!-- Source: iii section 70. -->
## Deprecation

Routes and fields can be marked (Candidate 2: spelled `#[rjango::deprecated(...)]` to avoid colliding with Rust's built-in attribute):

```
#[deprecated(
    since = "2.3",
    remove = "3.0",
    replacement = "..."
)]
```

Metadata propagates to:

```
OpenAPI
docs
generated SDK
MCP
rjango check
```
