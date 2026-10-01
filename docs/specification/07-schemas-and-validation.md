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


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (H10, M6, M7, M9, M1). Syntax is illustrative; claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment and retained sketches below.

### Derived schema projections: explicit, not repetitive

Models never implement input or output schemas themselves. A schema may be **derived from a model by an explicit field list**, the Rjango counterpart of DRF's `ModelSerializer` (illustrative):

```
#[rjango::schema(from = Venue, output, fields(id, slug, name, description, created_at))]
pub struct VenueResponse;

#[rjango::schema(from = Venue, input, fields(slug, name, description))]
pub struct CreateVenue;

#[rjango::schema(from = Venue, patch, fields(name, description))]
pub struct UpdateVenue;
```

- The macro generates the struct fields with their database-to-wire type mapping, validators implied by model constraints (length, range, enum), and the conversions:
  - `From<&Venue>` for output;
  - `Into<NewVenue>` for input, which fails to compile if required model fields are missing;
  - `Into<VenuePatch>` for patch.
- Field names are checked at compile time against `Venue::F`.
- **How a derive sees another type.** A schema macro sees only its own struct tokens, not `Venue`'s fields. Derivation therefore works through trait indirection: the model macro emits per-field associated types and constants (type, nullability, constraints, wire mapping) reachable as `<Venue as ModelFields>::...`, and the schema macro generates code that names them. A misspelled field then surfaces as an "unknown field" error. It must be made readable with `#[diagnostic::on_unimplemented]` on the field-lookup traits plus a macro-emitted span on the field list; the message quality is VALIDATION REQUIRED ([24](24-developer-experience-and-diagnostics.md) rule 7).
- Additional fields, renames, `#[sensitive]` and custom validators can be added in the struct body.
- Adding a model field never adds it to a derived schema, so exposure stays explicit.
- `fields(*)` is deliberately not supported.

### One tri-state type

`Patch<T>` (`Absent`, `Null`, `Value(T)`) is the only three-state type. It is used by PATCH schemas and by the ORM's `VenuePatch` ([03](03-models-and-orm.md)); the ORM's `Change<T>` is SUPERSEDED. `Null` is only available for nullable targets.

### Validation boundaries

Schema validators are synchronous, pure functions of the input. Validation that needs I/O (uniqueness pre-checks, existence of referenced IDs, quotas) belongs to the operation, which has `ctx`. "Async validation should also be possible" below is SUPERSEDED for schema validators. Database constraints remain authoritative for uniqueness.

### Sensitive fields

The schema macro generates `Debug` for schemas containing `#[sensitive]` fields, redacting them. Adding `#[derive(Debug)]` alongside `#[sensitive]` is a compile error with a pointer to this rule. Serialization of `#[sensitive]` output fields requires an explicit `#[sensitive(expose)]` on the output schema.

### Wire numerics

- Application `i64`/`u64` default to decimal strings, as in the Candidate 1 table.
- Framework-owned counts (`Page.total`, `pages`, bulk `affected`) use the `Count` wire type: a JSON number guaranteed ≤ 2^53−1. Values beyond that saturate, and the enclosing object carries a sibling boolean (for example `total_saturated: true`) rather than overflowing.
- `i32`/`u32` and smaller are JSON numbers.
- Unsigned types are wire types only; see [02](02-application-metadata-graph.md).

### Macro forms

Attribute macros are the convention (`#[rjango::model]`, `#[rjango::schema]`). Options use `key = value` or bare flags (`#[validate(email, min_length = 2)]`). The `#[derive(Schema)]` and `#[min_length(2)]` spellings in sketches are illustrative variants, not alternatives the final API will support.

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

> **SUPERSEDED (Candidate 2):** schema validators are synchronous; validation that needs I/O belongs to the operation.

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
