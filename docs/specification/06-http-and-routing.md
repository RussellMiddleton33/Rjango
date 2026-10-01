# HTTP and routing

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III.

Axum and Tower provide the HTTP foundation. Rjango adds typed handlers, explicit context, route metadata, middleware policy, diagnostics, and consistent lifecycle behavior.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Shared operations and Problem Details

HTTP handlers adapt inputs to [shared operations](22-application-operations-and-services.md), then project explicit response schemas. Transport authentication/body limits complement operation authorization.

Default HTTP API errors follow [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457.html), normally application/problem+json. Rjango's profile uses stable absolute type URIs, title, matching HTTP status, safe occurrence detail and optional non-sensitive instance. Stable code, correlation ID and field-validation errors are documented extensions. SQL, causes, secrets and stack traces remain private. Clients dispatch by type/code, tolerate unknown extensions and never parse localized prose. CLI/MCP project the same canonical error descriptors into their transport formats. Bodyless responses and failures after streaming begins require explicit transport handling.


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (C2, H2, H3, M1, M3). Syntax is illustrative; claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment and retained sketches below.

- **Handlers are implicit operations.** An annotated handler is an operation with the defaults in [22](22-application-operations-and-services.md): a declared policy or `public`, a scoped `Db`, command shielding, and Problem Details errors. Nothing routes around the operation contract. Escape-hatch Axum routes mounted directly are reported by `rjango check` as unmanaged, carry no AMG policy, and require an explicit `unmanaged_routes` allowance per app.
- **Disconnects.** Disconnect and timeout cancel Query handlers. Command handlers run to completion or deadline ([01](01-runtime-architecture.md)). The "Cancellation" section below is corrected accordingly.
- **Paths.** Path syntax is `/{param}` (Axum 0.8). Sketches using `/:param` are SUPERSEDED.
- **Errors.** The "HTTP Errors" envelope (`{"error": {...}}`) below is SUPERSEDED by RFC 9457 Problem Details. The error model and its status mapping (scoped not-found is 404; referenced-ID not-found is 422; unknown commit is 503) are in [24](24-developer-experience-and-diagnostics.md).
- **Diagnostics.** The route macro checks each extractor argument separately (body last) and asserts that the future is `Send` ([24](24-developer-experience-and-diagnostics.md)).
- **Return types.** Handlers return output schemas. Returning a model type is an error, because models do not implement output schemas.
- **Middleware order.** `RoutePermissions` in the ordering sketch below becomes `Exposure`: transport requirements only. Business policy runs inside the operation.
- **Request context.** `RequestContext` is replaced by the adapter-neutral `Ctx` from [22](22-application-operations-and-services.md). Request-only data (headers, client info) remains available to the handler through extractors.

## Open decisions and interpretation

Exact routing/middleware syntax and full proxy/deployment guidance remain open. Examples express contracts, not a usable framework API.



<!-- Source: iii section 30. -->
## HTTP Architecture

Rjango's HTTP system sits above Axum/Tower without hiding them.

```
HTTP
 │
 ▼
Hyper/Axum
 │
 ▼
Tower stack
 │
 ▼
Rjango HTTP abstraction
 │
 ▼
Application handler
```

Axum already provides typed state, extractors, response conversion and Tower integration, so Rjango should compose those strengths rather than rebuild them.

---

<!-- Source: iii section 31. -->
## Handler Design

Canonical:

```
#[rjango::get("/venues/{id}")]
async fn venue_detail(
    db: Db,
    Path(id): Path<Uuid>,
) -> Result<VenueResponse> {
    ...
}
```

Rjango should allow common arguments to be inferred as extractors.

Potential:

```
Json<CreateVenue>
Query<VenueFilter>
Path<Uuid>
Headers
Cookies
CurrentUser
Db
RequestContext
```

---

<!-- Source: iii section 32. -->
## Explicit Request Context

Provide:

```
RequestContext
```

containing safe request-scoped information (Candidate 2: superseded by the adapter-neutral `Ctx`; request-only data stays in extractors):

```
request ID
route ID
start time
authenticated identity
client information
trace span
extensions
```

It should not become a dumping ground for global state.

---

<!-- Source: iii section 33. -->
## Router Metadata

Every route contributes:

```
method
path
name
handler
input
output
permissions
middleware
version
tags
source location
```

to AMG.

Route naming should be stable:

```
route:venues.detail
```

---

<!-- Source: iii section 34. -->
## Nested Routers

Apps should be able to declare:

```
RjangoApp::new("venues")
    .prefix("/venues")
```

so:

```
/{id}
/search
```

becomes:

```
/venues/{id}
/venues/search
```

---

<!-- Source: iii section 35. -->
## Route Names

Every route should have a logical name independent of URL.

Example:

```
venues.detail
```

URL generation:

```
routes::venues::detail.url(id)
```

Potentially compile-time typed.

This prevents hard-coded URL strings throughout applications.

---

<!-- Source: iii section 36. -->
## Middleware

Three scopes:

```
Global

App

Route
```

Example:

```
global:
request ID
tracing
compression
security headers

app:
API-specific behavior

route:
authorization
rate limits
```

Tower remains the underlying interoperability model. Tower HTTP already supplies mature tracing, compression, timeout and related middleware primitives.

---

<!-- Source: iii section 37. -->
## Middleware Ordering

Ordering must be deterministic and introspectable.

Rjango should be able to show:

```
rjango inspect middleware
```

conceptually:

```
Request

RequestId
  ↓
TrustedProxy
  ↓
Tracing
  ↓
SecurityHeaders
  ↓
CORS
  ↓
Session
  ↓
Authentication
  ↓
RoutePermissions
  ↓
Handler

Response
```

Middleware ordering bugs are common enough that this deserves first-class tooling.

---

<!-- Source: iii section 38. -->
## Timeouts

Default request timeouts should exist in production configuration.

But streaming endpoints require different behavior.

Rjango distinguishes:

```
handler timeout

request-body timeout

response-body idle timeout
```

rather than pretending one timer handles every scenario.

Tower HTTP itself distinguishes request-future and body timing concerns, reinforcing the need for an explicit Rjango policy.

---

<!-- Source: iii section 39. -->
## Cancellation

> **Candidate 2 note:** this applies to Query handlers. Command handlers are shielded and run to completion or deadline ([01](01-runtime-architecture.md)).

When the client disconnects or a timeout occurs:

```
request future cancelled
```

Application code must understand that async work may be dropped.

Rjango documentation must explicitly cover:

- transaction cancellation
- spawned background work
- external API calls
- cleanup
- idempotency

A request handler must not spawn untracked durable work and assume it will finish.

Durable work belongs in the jobs system.

---

<!-- Source: iii section 40. -->
## Request Bodies

Support:

```
JSON
form-urlencoded
multipart
bytes
text
stream
```

Limits must be configurable globally and per-route.

Example:

```
#[body_limit = "10mb"]
```

Default limits should prevent accidental unbounded memory consumption.

---

<!-- Source: iii section 41. -->
## File Uploads

Uploads should support streaming.

Never require:

```
entire 2GB upload
→ memory
```

Rjango should allow:

```
stream → storage backend
```

with:

- maximum size
- MIME validation
- extension validation
- checksum
- filename sanitization
- temporary storage

---

<!-- Source: iii section 42. -->
## Responses

Handlers may return:

```
Result<T>
Json<T>
Html<T>
Redirect
File
Stream
NoContent
```

Domain/schema types with an appropriate Rjango response trait should serialize automatically.

---

<!-- Source: iii section 43. -->
## Streaming

First-class support for:

```
streaming downloads
SSE
large query responses
AI token streams
```

Backpressure must flow through underlying async streams.

---

<!-- Source: iii section 44. -->
## Content Negotiation

Initially:

```
JSON-first
```

Later:

```
HTML
MessagePack
other serializers
```

Rjango should not overcomplicate v1 with generic content negotiation unless actual use cases justify it.

---

<!-- Source: iii section 45. -->
## HTTP Errors

> **SUPERSEDED (Candidates 1 and 2):** the envelope below is replaced by RFC 9457 Problem Details; see the amendments above and [24](24-developer-experience-and-diagnostics.md).

Errors should use one canonical structured representation.

Example:

```
{
  "error": {
    "code": "validation_error",
    "message": "The request could not be validated.",
    "request_id": "req_...",
    "fields": {
      "email": [
        {
          "code": "invalid_email",
          "message": "Enter a valid email address."
        }
      ]
    }
  }
}
```

Production error responses never expose internal stack information.

---

<!-- Source: iii section 46. -->
## Request IDs

Every request receives an ID.

Used across:

```
HTTP response
logs
traces
errors
SQL telemetry
jobs spawned from request
audit trail
support diagnostics
```

The request ID should not itself contain sensitive information.

---

<!-- Source: iii section 47. -->
## Reverse Proxy Awareness

Production Rjango often sits behind:

```
Cloudflare
AWS ALB
nginx
Caddy
Kubernetes ingress
```

Forwarded headers must only be trusted when the proxy is explicitly configured.

Never blindly accept:

```
X-Forwarded-For
X-Forwarded-Proto
Forwarded
```

from arbitrary clients.

---

<!-- Source: iii section 48. -->
## Health Endpoints

Rjango should distinguish:

```
liveness
readiness
```

Liveness:

> Is the process functioning?

Readiness:

> Should this instance receive traffic?

Readiness may include:

```
database pool available
mandatory dependencies available
startup complete
```
