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

Routes and fields can be marked:

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
