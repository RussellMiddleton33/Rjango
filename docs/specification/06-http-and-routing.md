# HTTP and routing

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III.

Axum and Tower provide the HTTP foundation. Rjango adds typed handlers, explicit context, route metadata, middleware policy, diagnostics, and consistent lifecycle behavior.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

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

containing safe request-scoped information:

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
