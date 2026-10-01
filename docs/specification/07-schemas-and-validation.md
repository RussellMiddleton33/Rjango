# Schemas and validation

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III.

Persistence models, input schemas, and output schemas have separate responsibilities. Validation and serialization must preserve sensitivity, nested error paths, and absent/null/value semantics.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Database type system versus API wire type system

DatabaseType describes storage/constraints/precision/backend capability; WireType describes serialization/validation/client representation. Model fields never automatically become public schemas. Explicit mapping declares nullability versus omission, range, precision and timezone semantics. Database and API fingerprints evolve independently.

| Value | Default JSON / TypeScript contract |
| --- | --- |
| i64 / u64 | Decimal string / validated string, optionally branded. Number requires an explicit JavaScript-safe integer range. Bigint is an opt-in SDK conversion, not JSON. |
| Decimal | Precision-preserving decimal string / string or explicit decimal-library adapter; no implicit floating conversion. |
| DateTime | Offset-bearing RFC 3339 string / string; instant normalization/fractional precision declared. Date conversion is opt-in and documents precision loss. |
| Local date/time | Explicit local value; no invented UTC instant. Timezone interpretation is a separate contract. |

OpenAPI, codecs, validation and docs must agree. Test overflow, negative u64, precision loss, malformed dates and DST ambiguity. Backend unsigned-range support remains validation-required.


## Open decisions and interpretation

Detailed validator extension APIs, schema evolution compatibility, and exact derive/macro syntax remain open.



<!-- Source: iii section 49. -->
## Schemas & Serialization

Models describe persistence.

Schemas describe external/internal structured data.

These concepts must remain separate.

Example:

```
#[rjango::schema]
pub struct CreateUser {
    #[email]
    pub email: String,

    #[min_length = 2]
    #[max_length = 100]
    pub name: String,
}
```

---

<!-- Source: iii section 50. -->
## Schema Type Information

Schema metadata includes:

```
type
nullable
required
default
constraints
description
examples
serialization name
deprecation
sensitivity
```

This powers:

```
validation
OpenAPI
TypeScript
Admin
MCP
documentation
```

---

<!-- Source: iii section 51. -->
## Input vs Output

Explicit concepts:

```
InputSchema
OutputSchema
```

A type may implement both, but they are semantically distinct.

Example:

```
User database model

CreateUser input
UpdateUser input
UserResponse output
UserSummary output
```

This prevents accidental exposure of internal fields.

---

<!-- Source: iii section 52. -->
## Sensitive Fields

Schemas should support:

```
#[sensitive]
pub token: String
```

Sensitive metadata should affect:

```
logging
debug output
MCP
traces
error representation
documentation examples
```

It does not replace authorization.

---

<!-- Source: iii section 53. -->
## Validation

Built-in validators:

```
required
min/max
length
email
URL
UUID
regex
range
one_of
datetime constraints
```

Custom:

```
#[validate = validate_username]
```

Async validation should also be possible where necessary.

But distinguish:

```
pure schema validation
```

from:

```
database/business validation
```

---

<!-- Source: iii section 54. -->
## Validation Ordering

Recommended:

```
deserialize
   ↓
structural validation
   ↓
field validation
   ↓
schema validation
   ↓
business/service validation
   ↓
database operation
```

Unique constraints still require database enforcement because pre-checks alone are race-prone.

---

<!-- Source: iii section 55. -->
## Nested Schemas

Example:

```
pub struct VenueResponse {
    pub id: Uuid,
    pub name: String,
    pub address: AddressResponse,
    pub floors: Vec<FloorResponse>,
}
```

Metadata graph preserves nested relationships.

---

<!-- Source: iii section 56. -->
## Partial Updates

PATCH requires three-state semantics:

```
missing
value
null
```

Therefore:

```
Patch<T>
```

or equivalent must distinguish them explicitly.

Example:

```
description absent
→ don't modify

"description": null
→ set NULL

"description": "Hello"
→ set value
```

Using ordinary `Option<T>` alone cannot represent all three.

---

<!-- Source: iii section 57. -->
## Schema Versioning

Schemas have stable AMG IDs.

Example:

```
schema:users.UserResponse
```

API versioning may reference different schema definitions without mutating historical contracts.

---

<!-- Source: iii section 58. -->
## Serialization Naming

Allow:

```
#[json_name = "createdAt"]
pub created_at: DateTime<Utc>,
```

But project-wide naming conventions should be configurable:

```
snake_case
camelCase
```

Generated TypeScript must use actual serialized names.
