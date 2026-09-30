# Models and ORM

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** ORM specification.

Rjango owns its public model and query API; SeaORM 2.x is the selected ORM foundation over SQLx 0.9, with an explicit lower-level escape hatch. These are design targets, not a verified dependency matrix.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Open decisions and interpretation

The final locks and open decisions are preserved below. In particular, model macro form, `F`/`R` syntax, creation/builders, partial selections, loaded-state representation, and backend wrapping require evidence before syntax is frozen.

## Dependency and syntax caveats

The architecture selects SeaORM 2.x over SQLx 0.9 and PostgreSQL first. Exact release compatibility, shared pool/transaction interoperability, generated-code ergonomics and performance remain VALIDATION REQUIRED. No dependency manifest is introduced by this capture.

The original “PROVISIONALLY VALIDATED” label meant an ecosystem review, not a successful test run. It is normalized to the shared status legend. Benchmark matrices and acceptance examples below are existing evidence requirements, not an implementation schedule. The early implementation order (ORM §95) is SUPERSEDED and omitted.

<!-- Source: orm section 1. -->
## Objective

Rjango's data layer should provide the productivity of Django's ORM without disguising Rust as Python.

The developer should be able to think:

```
Model
Query
Relationship
Migration
Transaction
```

rather than:

```
Entity
Column enum
ActiveModel
SeaQuery expression
ConnectionTrait
Statement
```

for normal application development.

At the same time, an experienced Rust developer must be able to drop down into:

```
SeaORM
SeaQuery
SQLx
raw SQL
PostgreSQL
```

whenever needed.

---

<!-- Source: orm section 2. -->
## Architectural Stack

```
Application Code
      │
      ▼
 Rjango Models
      │
      ├────────────► Application Metadata Graph
      │
      ▼
 Rjango Query API
      │
      ▼
   SeaORM 2
      │
      ▼
   SeaQuery
      │
      ▼
    SQLx
      │
      ▼
 PostgreSQL
```

This distinction is extremely important.

### Rjango owns

- public model syntax
- public query API
- field metadata
- relationship metadata
- migration policy
- Django-style ergonomics
- errors
- diagnostics
- N+1 warnings
- instrumentation
- documentation
- TypeScript/OpenAPI metadata
- MCP representation
- compatibility guarantees

### SeaORM owns

- ORM machinery
- SQL construction
- entity mechanics
- relationship loading
- persistence internals
- database abstraction

### SQLx owns

- drivers
- connections
- pooling
- transactions
- asynchronous database I/O
- raw SQL
- compile-checked SQL escape hatch

---

<!-- Source: orm section 3. -->
## Critical Rule: Rjango Owns Its Public API

Application developers should not need to write SeaORM types for ordinary Rjango code.

Wrong architectural direction:

```
use sea_orm::*;

Entity::find()
    .filter(Column::Name.eq(...))
```

Rjango should instead present:

```
use rjango::prelude::*;

Venue::objects(&db)
    .filter(Venue::F.slug.eq("disney-springs"))
    .all()
    .await?;
```

Internally this can compile down to SeaORM.

This allows us to upgrade or potentially replace internal components without making every Rjango application rewrite its data layer.

---

<!-- Source: orm section 4. -->
## Explicit Database Context

One Django behavior we should **not** copy is ambient/global database access.

Do not design:

```
User::objects().all().await?
```

if it secretly discovers a global database connection.

Prefer:

```
User::objects(&db)
    .all()
    .await?;
```

Typical handler:

```
#[get("/venues")]
async fn venues(db: Db) -> Result<Json<Vec<Venue>>> {
    let venues = Venue::objects(&db)
        .all()
        .await?;

    Ok(Json(venues))
}
```

Reasons:

- explicit dependencies
- easier testing
- transaction correctness
- multiple database support
- tenant-specific databases later
- no task-local magic
- easier concurrency reasoning

`Db` should be a cheap clonable handle around the application database/pool.

---

<!-- Source: orm section 5. -->
## Model Declaration

I recommend an **attribute macro** rather than merely:

```
#[derive(Model)]
```

Canonical design:

```
#[rjango::model]
pub struct Venue {
    #[primary_key]
    pub id: Uuid,

    #[unique]
    #[index]
    pub slug: String,

    #[max_length = 200]
    pub name: String,

    pub description: Option<String>,

    #[default = true]
    pub active: bool,

    #[auto_now_add]
    pub created_at: DateTime<Utc>,

    #[auto_now]
    pub updated_at: DateTime<Utc>,
}
```

Why an attribute macro?

Because Rjango needs to generate considerably more than one trait implementation:

```
SeaORM representation
field descriptors
typed field handles
model metadata
insert type
change tracking type
table metadata
migration metadata
serialization hooks
MCP information
source references
```

An attribute macro gives us more control over generated code and better diagnostics.

---

<!-- Source: orm section 6. -->
## Generated Model Components

The previous declaration conceptually creates:

```
Venue
VenueFields
NewVenue
VenueChanges
VenueEditor
VenueQuery
VenueMetadata
hidden SeaORM representation
```

Application developers normally see only the useful portions.

---

<!-- Source: orm section 7. -->
## Fields API

We need typed field references.

Recommended syntax:

```
Venue::F.slug
Venue::F.name
Venue::F.created_at
```

Where the generated structure behaves conceptually like:

```
pub struct VenueFields {
    pub id: Field<Venue, Uuid>,
    pub slug: Field<Venue, String>,
    pub name: Field<Venue, String>,
    pub description: Field<Venue, Option<String>>,
    pub active: Field<Venue, bool>,
}
```

`F` is a generated constant:

```
Venue::F
```

This gives us concise syntax while maintaining Rust typing.

---

<!-- Source: orm section 8. -->
## Typed Queries

Examples:

```
Venue::objects(&db)
    .filter(Venue::F.active.eq(true))
    .all()
    .await?;
```

String operation:

```
Venue::objects(&db)
    .filter(Venue::F.name.contains("Disney"))
    .all()
    .await?;
```

PostgreSQL case-insensitive search:

```
Venue::objects(&db)
    .filter(Venue::F.name.ilike("%disney%"))
    .all()
    .await?;
```

Comparison:

```
Event::objects(&db)
    .filter(Event::F.starts_at.gt(now))
    .all()
    .await?;
```

The compiler should reject nonsense such as:

```
Venue::F.created_at.contains("Disney")
```

or:

```
Venue::F.active.gt("hello")
```

whenever possible.

---

<!-- Source: orm section 9. -->
## QuerySet Mental Model

`objects()` returns:

```
QuerySet<'db, Venue>
```

A QuerySet is:

- lazy
- typed
- composable
- immutable in meaning
- asynchronously executed only at terminal operations

Example:

```
let query = Venue::objects(&db)
    .filter(Venue::F.active.eq(true))
    .order_by(Venue::F.name.asc());
```

No database query yet.

Then:

```
let venues = query.all().await?;
```

executes it.

---

<!-- Source: orm section 10. -->
## QuerySet Operations

Initial target:

```
filter
exclude
order_by
limit
offset
select
with
distinct
group_by
having
lock
```

Terminal methods:

```
all
one
first
last
get
require
count
exists
stream
paginate
```

---

<!-- Source: orm section 11. -->
## Primary-Key Convenience

Django-like simplicity:

```
let venue = Venue::objects(&db)
    .get(id)
    .await?;
```

Behavior:

```
found      → Venue
not found  → Error::NotFound
```

For optional lookup:

```
let venue = Venue::objects(&db)
    .find(id)
    .await?;
```

returns:

```
Option<Venue>
```

This distinction should be extremely clear.

---

<!-- Source: orm section 12. -->
## `first()` Semantics

```
let venue = Venue::objects(&db)
    .filter(Venue::F.active.eq(true))
    .first()
    .await?;
```

returns:

```
Option<Venue>
```

For required value:

```
let venue = Venue::objects(&db)
    .filter(...)
    .require()
    .await?;
```

returns a model or `NotFound`.

---

<!-- Source: orm section 13. -->
## Complex Filtering

Support boolean composition:

```
Venue::objects(&db)
    .filter(
        Venue::F.active.eq(true)
            .and(Venue::F.name.contains("Center"))
    )
    .all()
    .await?;
```

OR:

```
.filter(
    Venue::F.city.eq("Atlanta")
        .or(Venue::F.city.eq("Dallas"))
)
```

NOT:

```
.filter(
    Venue::F.archived.eq(true).not()
)
```

---

<!-- Source: orm section 14. -->
## Django Q-Object Equivalent

Potential syntax:

```
query!(
    Venue::F.active == true &&
    (
        Venue::F.city == "Atlanta" ||
        Venue::F.city == "Dallas"
    )
)
```

This is attractive but should **not** be part of v0.1 until macro ergonomics and error quality have been tested.

Initial implementation should favor ordinary typed Rust methods.

---

<!-- Source: orm section 15. -->
## Creation

We should avoid partially initialized persisted models.

Generated:

```
pub struct NewVenue {
    pub slug: String,
    pub name: String,
    pub description: Option<String>,
}
```

Fields with:

```
database defaults
auto_now_add
generated IDs
```

do not have to be supplied.

Creation:

```
let venue = Venue::objects(&db)
    .create(NewVenue {
        slug: "disney-springs".into(),
        name: "Disney Springs".into(),
        description: None,
    })
    .await?;
```

---

<!-- Source: orm section 16. -->
## Builder Creation

Convenience:

```
let venue = Venue::objects(&db)
    .build()
    .slug("disney-springs")
    .name("Disney Springs")
    .create()
    .await?;
```

This should only ship if we can produce excellent compile-time/runtime messages for missing required fields.

We should prototype two approaches:

```
A. generated NewVenue struct
B. builder
```

and measure:

- ergonomics
- compile errors
- compile time
- generated code size

The generated struct is the safer v0.1 default.

---

<!-- Source: orm section 17. -->
## Updates

Avoid Django-style arbitrary mutation followed by magical dirty detection unless we can make behavior completely clear.

Recommended:

```
let venue = Venue::objects(&db)
    .get(id)
    .await?;

let venue = venue
    .edit()
    .name("Disney Springs Resort")
    .description("Updated")
    .save(&db)
    .await?;
```

Underneath, this can leverage SeaORM's change tracking.

Only changed columns should be updated.

---

<!-- Source: orm section 18. -->
## Explicit Changesets

Bulk/API updates need another abstraction.

Generated:

```
VenueChanges
```

But normal `Option<T>` is insufficient because nullable columns require distinguishing:

```
unchanged

set value

set NULL
```

Therefore Rjango should define:

```
pub enum Change<T> {
    Unchanged,
    Set(T),
    Null,
}
```

For nullable fields.

Usage:

```
VenueChanges {
    name: Change::Set("New name".into()),
    description: Change::Null,
    ..Default::default()
}
```

This avoids subtle PATCH bugs.

---

<!-- Source: orm section 19. -->
## Bulk Update

```
Venue::objects(&db)
    .filter(Venue::F.active.eq(false))
    .update(
        VenueChanges::default()
            .archived(true)
    )
    .await?;
```

Return:

```
UpdateResult {
    affected: u64
}
```

---

<!-- Source: orm section 20. -->
## Delete

Instance:

```
venue.delete(&db).await?;
```

Query:

```
Venue::objects(&db)
    .filter(Venue::F.archived.eq(true))
    .delete()
    .await?;
```

Destructive bulk operations should integrate with diagnostics.

Potential development warning:

```
RJG-DB-042

Bulk DELETE issued without restrictive filter.
```

---

<!-- Source: orm section 21. -->
## Relationships

Example:

```
#[rjango::model]
pub struct Floor {
    #[primary_key]
    pub id: Uuid,

    pub venue_id: Uuid,

    #[belongs_to(
        Venue,
        foreign_key = venue_id,
        on_delete = Cascade
    )]
    pub venue: BelongsTo<Venue>,

    pub name: String,
}
```

Inverse:

```
#[rjango::model]
pub struct Venue {
    ...

    #[has_many(Floor, foreign_key = venue_id)]
    pub floors: HasMany<Floor>,
}
```

---

<!-- Source: orm section 22. -->
## Relationship Types

Required:

```
BelongsTo<T>
HasOne<T>
HasMany<T>
ManyToMany<T>
```

The AMG translates these into:

```
RelationshipDescriptor
```

and explicit graph edges.

---

<!-- Source: orm section 23. -->
## Loading Relationships

Canonical Rjango syntax:

```
let venues = Venue::objects(&db)
    .with(Venue::R.floors)
    .all()
    .await?;
```

Where:

```
Venue::R
```

is generated relationship metadata analogous to:

```
Venue::F
```

for fields.

Potential:

```
F = fields
R = relationships
```

This is concise and readable.

---

<!-- Source: orm section 24. -->
## Nested Loading

```
Venue::objects(&db)
    .with(
        Venue::R.floors
            .with(Floor::R.units)
    )
    .all()
    .await?;
```

Rjango should defer relationship execution strategy to underlying proven machinery where possible.

The framework should choose between:

```
JOIN
batched relation query
data loader
```

rather than asking ordinary application developers to reason about each case.

---

<!-- Source: orm section 25. -->
## N+1 Policy

Rjango should make accidental N+1 behavior difficult.

### Rule

Normal relationship access should never silently execute an unbounded query per model.

Possible behavior:

```
venue.floors
```

when unloaded should produce something like:

```
RelationState::NotLoaded
```

rather than secretly querying.

Developers must explicitly request loading:

```
.with(Venue::R.floors)
```

or:

```
venue.load(Venue::R.floors, &db).await?
```

No invisible database I/O from property access.

---

<!-- Source: orm section 26. -->
## Development N+1 Detection

Rjango tracing should associate SQL statements with:

```
request
route
model
relationship
source location
```

Then detect patterns such as:

```
SELECT floor WHERE venue_id = ?
SELECT floor WHERE venue_id = ?
SELECT floor WHERE venue_id = ?
SELECT floor WHERE venue_id = ?
...
```

Potential warning:

```
RJG-PERF-001

Possible N+1 query detected.

Route:
GET /venues

Relationship:
venues.Venue.floors

Observed:
51 similar queries

Suggestion:
.with(Venue::R.floors)
```

This could become a major framework advantage.

---

<!-- Source: orm section 27. -->
## Transactions

First-class transactions:

```
db.transaction(|tx| async move {
    let organization =
        Organization::objects(tx)
            .create(new_org)
            .await?;

    User::objects(tx)
        .create(new_user)
        .await?;

    Ok(organization)
})
.await?;
```

Everything accepting a database should accept a common abstraction representing:

```
pool connection
transaction
```

without changing model APIs.

---

<!-- Source: orm section 28. -->
## Nested Transactions

Where supported, nested transaction semantics should map to savepoints.

Example:

```
tx.transaction(|savepoint| async move {
    ...
})
.await?;
```

Behavior must be explicitly documented and tested.

---

<!-- Source: orm section 29. -->
## Transaction Cancellation

This deserves specific tests.

If an async task is cancelled:

```
before COMMIT
```

Rjango must ensure transaction state is not accidentally committed.

Connection reuse must remain safe.

This belongs in our mandatory concurrency/failure test suite.

---

<!-- Source: orm section 30. -->
## Raw SQL Escape Hatch

Rjango must never make advanced SQL painful.

Example:

```
let rows = db
    .sqlx()
    .query_as::<CustomResult>(...)
    .fetch_all()
    .await?;
```

or equivalent safe adapter.

Rjango should make the underlying connection available intentionally.

---

<!-- Source: orm section 31. -->
## Compile-Checked SQL

Users should be able to use SQLx's compile-checked query macros where appropriate.

Example concept:

```
let result = rjango::sqlx::query!(
    r#"
    SELECT id, name
    FROM venues
    WHERE active = true
    "#
)
.fetch_all(db.sqlx())
.await?;
```

Rjango should not cripple SQLx's advanced functionality.

---

<!-- Source: orm section 32. -->
## PostgreSQL-First Strategy

Initial Rjango should target PostgreSQL exceptionally well.

Support:

```
UUID
JSONB
arrays
INET
timestamps
numeric
enums
full-text search
transactions
RETURNING
LISTEN/NOTIFY later
```

We should avoid artificially reducing PostgreSQL to the lowest common denominator.

---

<!-- Source: orm section 33. -->
## Database Portability

Core model metadata should remain portable where sensible.

Future:

```
SQLite
MySQL/MariaDB
```

But Rjango v0.x should not compromise PostgreSQL quality merely to advertise three database logos.

Database-specific features should be clearly namespaced.

Example:

```
#[postgres(jsonb)]
pub metadata: JsonValue
```

---

<!-- Source: orm section 34. -->
## Connection Pool

Rjango should configure and own a pool.

Example configuration:

```
[database]
url_env = "DATABASE_URL"

min_connections = 2
max_connections = 20

connect_timeout = "5s"
acquire_timeout = "5s"
idle_timeout = "10m"
```

Defaults should be safe but configurable.

---

<!-- Source: orm section 35. -->
## Pool Instrumentation

Expose:

```
connections open
connections idle
connection acquisition latency
timeouts
query duration
```

to tracing/metrics.

The runtime plane can associate pool health with:

```
database:default
```

---

<!-- Source: orm section 36. -->
## Query Instrumentation

Every query should carry metadata when practical:

```
model
operation
route
duration
rows
```

Example:

```
db.query

model = venues.Venue
operation = SELECT
duration = 7.2ms
rows = 38
```

Never log bound sensitive values by default.

---

<!-- Source: orm section 37. -->
## Slow Query Detection

Development:

```
RJG-PERF-010

Slow database query

Model:
venues.Venue

Duration:
427ms

Route:
GET /venues/search

SQL fingerprint:
...
```

Threshold configurable.

---

<!-- Source: orm section 38. -->
## Database Errors

SeaORM/SQLx errors should not leak through ordinary application APIs.

Rjango maps them into:

```
pub enum DatabaseError {
    Connection,
    Timeout,
    UniqueViolation,
    ForeignKeyViolation,
    NotNullViolation,
    SerializationFailure,
    Deadlock,
    ConstraintViolation,
    Query,
    Unknown,
}
```

Application code can safely match:

```
DatabaseError::UniqueViolation { constraint, .. }
```

without parsing PostgreSQL strings.

---

<!-- Source: orm section 39. -->
## Constraint Mapping

AMG IDs should connect database constraints to model metadata.

Example:

```
constraint:users.User.email_unique
```

Database violation:

```
users_email_key
```

maps back to:

```
field:users.User.email
```

Then API validation can produce:

```
{
  "field": "email",
  "code": "unique",
  "message": "A user with this email already exists."
}
```

This is much better than exposing raw database errors.

---

<!-- Source: orm section 49. -->
## Metadata Integration

Every model contributes:

```
ModelDescriptor
FieldDescriptor
RelationshipDescriptor
IndexDescriptor
ConstraintDescriptor
```

to AMG.

Example:

```
model:venues.Venue
    │
    ├── field:venues.Venue.id
    ├── field:venues.Venue.name
    ├── field:venues.Venue.slug
    │
    └── relation:venues.Venue.floors
```

---

<!-- Source: orm section 50. -->
## MCP

An AI should be able to ask:

```
inspect_model("venues.Venue")
```

and receive:

```
{
  "id": "model:venues.Venue",
  "table": "venues",
  "primary_key": "id",
  "fields": [],
  "relationships": [],
  "indexes": [],
  "constraints": []
}
```

without reading source code.

---

<!-- Source: orm section 51. -->
## MCP Query Inspection

Potential read-only tooling:

```
inspect_model
inspect_relationships
inspect_indexes
inspect_migration_history
inspect_pending_model_changes
```

Development-only privileged tools:

```
generate_migration
validate_migration
run_migration
```

`run_migration` must be independently permissioned.

---

<!-- Source: orm section 52. -->
## MCP Database Access Is Separate

Important:

```
Metadata access
≠
database data access
```

Agent may know:

```
User.email exists
```

without being allowed to read user email values.

Permissions:

```
[mcp.permissions]
schema.read = true
database.read = false
database.write = false
```

---

<!-- Source: orm section 58. -->
## Pagination

Built-in:

```
let page = Venue::objects(&db)
    .filter(Venue::F.active.eq(true))
    .order_by(Venue::F.name.asc())
    .paginate(PageRequest {
        number: 1,
        size: 25,
    })
    .await?;
```

Result:

```
Page<Venue>
```

containing:

```
items
page
page_size
total
pages
has_next
has_previous
```

Cursor pagination should also be first-class.

---

<!-- Source: orm section 59. -->
## Streaming

Large result sets:

```
let mut venues = Venue::objects(&db)
    .stream()
    .await?;

while let Some(venue) = venues.next().await {
    ...
}
```

Must not load all records into memory.

---

<!-- Source: orm section 60. -->
## Aggregation

Target:

```
Order::objects(&db)
    .filter(Order::F.status.eq(Status::Complete))
    .sum(Order::F.total)
    .await?;
```

Also:

```
count
sum
avg
min
max
```

---

<!-- Source: orm section 61. -->
## Partial Selections

Avoid loading full records when unnecessary.

```
let names = Venue::objects(&db)
    .select((Venue::F.id, Venue::F.name))
    .all()
    .await?;
```

Return type should remain typed.

Implementation design needs a prototype because Rust tuple/generic ergonomics can become ugly rapidly.

---

<!-- Source: orm section 62. -->
## Custom Query DTOs

Preferred advanced pattern:

```
#[derive(QueryResult)]
struct VenueSummary {
    id: Uuid,
    name: String,
    floor_count: i64,
}
```

Then:

```
Venue::objects(&db)
    .select_as::<VenueSummary>()
    ...
```

---

<!-- Source: orm section 63. -->
## Model Hooks

Be conservative.

Potential hooks:

```
before_insert
after_insert
before_update
after_update
before_delete
after_delete
```

But they should not become Django signals with hidden global behavior.

Prefer explicit domain events for cross-component effects.

---

<!-- Source: orm section 64. -->
## Domain Events

Eventually:

```
#[event]
pub struct VenueCreated {
    pub venue_id: Uuid,
}
```

This is preferable to invisible signal chains for important business behavior.

Database lifecycle hooks should stay local and predictable.

---

<!-- Source: orm section 65. -->
## Soft Delete

Do not make soft deletion magic.

Optional:

```
#[soft_delete(field = deleted_at)]
```

would cause normal QuerySets to exclude deleted records.

But expose explicitly:

```
Venue::objects(&db).with_deleted()
```

and:

```
Venue::objects(&db).only_deleted()
```

This should be an extension, not core v0.1.

---

<!-- Source: orm section 66. -->
## Multi-Tenancy

Do not bake multi-tenancy deeply into initial ORM.

But the explicit DB/query context should make future options possible:

```
shared database / tenant column
schema-per-tenant
database-per-tenant
```

This is another reason to avoid hidden global connection state.

---

<!-- Source: orm section 67. -->
## Optimistic Locking

Potential model feature:

```
#[version]
pub version: i64
```

Update:

```
UPDATE ...
WHERE id = ?
AND version = ?
```

and increment.

Conflicts become:

```
ConcurrentModification
```

Worth supporting later.

---

<!-- Source: orm section 68. -->
## Database Generated IDs

Defaults should support:

```
UUID
serial / identity
custom
```

Recommended modern default:

```
#[primary_key]
#[default = uuid_v7]
pub id: Uuid,
```

Exact default policy should be separately tested before being framework-wide.

---

<!-- Source: orm section 69. -->
## Enums

Example:

```
#[rjango::enum]
pub enum VenueStatus {
    Draft,
    Active,
    Archived,
}
```

Usage:

```
pub status: VenueStatus
```

Metadata feeds:

```
database
OpenAPI
TypeScript
admin dropdowns
MCP
```

---

<!-- Source: orm section 70. -->
## Custom Types

Provide a trait:

```
pub trait DatabaseValue {
    ...
}
```

with AMG metadata mapping.

We should not require developers to wait on Rjango whenever PostgreSQL adds or an application needs an unusual type.

---

<!-- Source: orm section 71. -->
## Compile-Time Diagnostics

These should fail compilation cleanly:

```
multiple primary keys without composite declaration
relationship references unknown field
nullable relation backed by non-null FK
invalid default type
auto_now on incompatible field
unsupported field type
duplicate Rust-level field metadata
```

Use `trybuild`-style compile-pass/compile-fail tests.

---

<!-- Source: orm section 72. -->
## Runtime Validation

Some things cannot be proven until startup/database connection.

Examples:

```
database reachable
migration state
database schema drift
unsupported database extension
pool configuration
```

These feed:

```
rjango check
```

---

<!-- Source: orm section 73. -->
## Required Public API Tests

Every public model/query operation requires tests for:

```
normal success
empty result
invalid input
database failure
constraint failure
transaction failure
cancellation where relevant
```

---

<!-- Source: orm section 74. -->
## Coverage Standard

Rjango-owned testable code:

```
100% line coverage target
100% meaningful branch coverage where practical
```

No unexplained exclusions.

Any exclusion must include:

```
reason
owner
scope
why practical testing is impossible
```

---

<!-- Source: orm section 75. -->
## Coverage Is Not Enough

Required supplementary validation:

```
unit tests
PostgreSQL integration tests
compile-fail tests
property tests
fuzz tests
mutation tests
concurrency tests
cancellation tests
snapshot tests
migration round-trip tests
compatibility tests
performance benchmarks
security tests
documentation tests
```

---

<!-- Source: orm section 76. -->
## ORM Prototype Gate

Before locking the public ORM API, build the following real prototype.

Models:

```
Organization
User
Venue
Floor
Unit
Tag
VenueTag
```

It must cover:

```
1-1
1-N
N-1
N-N
nullable foreign key
composite unique constraint
JSONB
enum
timestamps
UUID PK
indexes
transaction
```

---

<!-- Source: orm section 77. -->
## Query Prototype Gate

The prototype must prove:

```
Venue::objects(&db)
    .filter(Venue::F.active.eq(true))
    .with(Venue::R.floors)
    .order_by(Venue::F.name.asc())
    .all()
    .await?;
```

works cleanly.

Also compile-test invalid examples.

---

<!-- Source: orm section 78. -->
## SQL Inspection Tests

For every major query pattern, inspect generated SQL.

Compare:

```
Rjango
SeaORM directly
SQLx/direct SQL baseline
```

Verify:

```
query count
JOINs
bind parameters
selected columns
ORDER BY
LIMIT
indexes usable
```

---

<!-- Source: orm section 79. -->
## Performance Benchmark Matrix

Benchmark:

```
A. Raw SQLx
B. Direct SeaORM
C. Rjango
```

Operations:

```
single PK lookup
100-row list
1000-row list
insert
batch insert
update
transaction
1-1 loading
1-N loading
nested relationships
pagination
streaming
```

Metrics:

```
throughput
p50
p95
p99
allocations
memory
CPU
query count
```

---

<!-- Source: orm section 80. -->
## Performance Acceptance Goal

Rjango should not impose meaningful unexpected overhead over SeaORM.

We should define the exact threshold only after baseline measurement rather than inventing a percentage now.

Any significant difference must be explained.

---

<!-- Source: orm section 81. -->
## N+1 Verification

Build intentionally broken relational access tests.

Example data:

```
100 venues
10 floors per venue
```

Desired:

```
Venue + floors
```

should execute a bounded number of queries rather than:

```
1 + 100
```

We should assert query count directly.

---

<!-- Source: orm section 82. -->
## Concurrency Test

Spawn large numbers of concurrent operations.

Example:

```
1,000 tasks
mixed reads/writes
shared pool
transactions
```

Verify:

```
no deadlocks
no leaked connections
no invalid shared state
bounded memory
correct data
```

---

<!-- Source: orm section 83. -->
## Cancellation Test

Cancel queries and transactions at different points:

```
waiting for pool
waiting for query
inside transaction
before commit
```

Verify the pool and transaction state recover correctly.

---

<!-- Source: orm section 84. -->
## Migration Round-Trip Tests

For each supported migration operation:

```
Schema A
  ↓
migration
  ↓
Schema B
  ↓
reverse
  ↓
Schema A
```

Verify structural equality.

Where a destructive migration cannot be losslessly reversed, document and test that behavior.

---

<!-- Source: orm section 85. -->
## Property Tests

Generate random valid model graphs.

Test properties:

```
serialization deterministic
fingerprints deterministic
schema diff stable
migration ordering valid
foreign keys resolve
dependency ordering acyclic where required
```

---

<!-- Source: orm section 86. -->
## Fuzzing

Fuzz:

```
metadata parsers
migration graph processing
config parsing
raw identifier handling
SQL identifier quoting
relationship resolution
```

Especially test attacker-controlled identifiers and unexpected Unicode.

---

<!-- Source: orm section 87. -->
## Mutation Testing

Critical modules:

```
migration safety classification
permission-aware database tooling
constraint mapping
query filters
metadata diff
```

Tests should kill mutations.

If flipping:

```
safe → destructive
```

or:

```
eq → ne
```

doesn't fail a test, the test suite isn't strong enough.

---

<!-- Source: orm section 88. -->
## Documentation Tests

Every documentation example intended to compile should be executed by CI.

Examples in:

```
README
tutorial
ORM guide
Django migration guide
API docs
examples/
```

must not rot.

---

<!-- Source: orm section 89. -->
## Human Documentation Requirement

No ORM feature is done without:

```
concept overview
quick-start example
API reference
common patterns
edge cases
error explanations
performance guidance
Django equivalent
troubleshooting
```

---

<!-- Source: orm section 90. -->
## AI Documentation Requirement

Every ORM capability should also produce structured agent-facing knowledge.

An AI should be able to answer:

```
How do I filter a model?

How do relationships work?

What should I use instead of select_related?

How do I perform a transaction?

How do I write raw SQL?

Why is this query producing an N+1 warning?
```

against the installed Rjango version.

---

<!-- Source: orm section 91. -->
## Documentation Version Awareness

Agent guidance must know:

```
Rjango version
metadata schema version
feature availability
deprecations
```

An agent must not recommend v0.8 syntax to a v0.6 project.

---

<!-- Source: orm section 92. -->
## Error Documentation

Every Rjango error code gets a permanent documentation page.

Example:

```
RJG-DB-021
```

CLI output:

```
For details:

rjango explain RJG-DB-021
```

MCP:

```
explain_error("RJG-DB-021")
```

Human website:

```
errors/RJG-DB-021
```

One canonical explanation powers all three.

---

<!-- Source: orm section 93. -->
## Stability Rule

Rjango public types must not expose SeaORM implementation-specific types unless inside an explicitly documented escape-hatch namespace.

Good:

```
rjango::db::QuerySet
```

Escape hatch:

```
rjango::db::seaorm()
```

Avoid:

```
pub fn objects() -> sea_orm::Select<...>
```

because it locks Rjango's public API to SeaORM.

---

<!-- Source: orm section 94. -->
## Initial Crates

Recommended:

```
rjango-db
    connection abstraction
    transactions
    errors
    instrumentation

rjango-models
    model macro
    field system
    relationships
    queryset

rjango-migrations
    AMG diff
    operations
    migration files
    runner

rjango-macros
    procedural macros
```

---

<!-- Source: orm section 96. -->
## Evidence status

**Decision: SPEC-LOCKED. Evidence: VALIDATION REQUIRED.**

The original conversation used “PROVISIONALLY VALIDATED” for an ecosystem review and proposed a progression through research, prototypes, and benchmarks. That label did not mean the prototypes ran. Under the shared status legend, architecture agreement and validation evidence are tracked separately. No implementation, compatibility result, benchmark, or passing test is established by this documentation capture. Move a claim to VALIDATED only with a linked, reproducible result and its tested versions and scope.

---

<!-- Source: orm section 97. -->
## Decisions We Are Comfortable Locking Now

#### LOCK

Async-first database access.

#### LOCK

PostgreSQL-first.

#### LOCK

Explicit database context.

#### LOCK

Rjango owns public ORM API.

#### LOCK

SeaORM 2.x is initial ORM engine.

#### LOCK

SQLx remains accessible as escape hatch.

#### LOCK

Models feed the AMG.

#### LOCK

Production changes use explicit migrations.

#### LOCK

Relationship access must not cause invisible per-row database queries.

#### LOCK

Queries are instrumentable.

#### LOCK

100% practically testable framework-owned coverage target.

#### LOCK

Human + AI + Django-transition documentation ship with features.

---

<!-- Source: orm section 98. -->
## Decisions That Must Remain Open Until Prototype Testing

#### OPEN

Exact `Venue::F.name` syntax.

#### OPEN

Exact relationship syntax.

#### OPEN

Attribute macro versus mixed attribute/derive strategy.

#### OPEN

Generated `NewVenue` versus builder-first creation.

#### OPEN

Partial-select type syntax.

#### OPEN

Exact relationship loaded-state representation.

#### OPEN

How much SeaORM functionality to wrap versus expose.

#### OPEN

Exact migration serialization format.

#### OPEN

Performance regression thresholds.

These should not be prematurely frozen.

---

<!-- Source: orm section 99. -->
## First Acceptance Prototype

The first ORM proof application should implement:

```
Organization
 └── Users

Venue
 ├── Floors
 │    └── Units
 │
 └── Tags
```

And demonstrate:

```
let venues = Venue::objects(&db)
    .filter(Venue::F.active.eq(true))
    .with(
        Venue::R.floors
            .with(Floor::R.units)
    )
    .order_by(Venue::F.name.asc())
    .all()
    .await?;
```

Then verify:

```
✓ type safety
✓ correct models
✓ bounded query count
✓ correct generated SQL
✓ transaction safety
✓ AMG metadata
✓ migration generation
✓ TypeScript-visible metadata
✓ MCP inspection
✓ human docs
✓ Django comparison docs
✓ compile-fail behavior
✓ coverage requirements
✓ benchmark overhead
```

If that test reveals that our public abstraction is fighting SeaORM or Rust, we change the public API before publishing it.

---

<!-- Source: orm section 100. -->
## North Star

A developer coming from Django should understand this immediately:

```
venues = (
    Venue.objects
    .filter(active=True)
    .prefetch_related("floors")
    .order_by("name")
)
```

Rjango:

```
let venues = Venue::objects(&db)
    .filter(Venue::F.active.eq(true))
    .with(Venue::R.floors)
    .order_by(Venue::F.name.asc())
    .all()
    .await?;
```

But the Rjango version additionally provides:

```
compile-time types
async execution
multi-thread-safe runtime
structured metadata
query instrumentation
automatic API schema knowledge
migration intelligence
N+1 diagnostics
MCP introspection
AI-agent understanding
```

The goal is not:

> Recreate Django ORM syntax in Rust.

The goal is:

> Take the mental model that makes Django productive and redesign it around the things Rust can guarantee.
