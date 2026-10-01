# Application Metadata Graph

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** AMG specification.

The AMG is an immutable, deterministic description of the application, derived from code and explicit registration. Definition metadata, runtime bindings, and operational state remain distinct.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Provenance and completeness

Relevant facts and edges carry Generated, Declared, Inferred or Observed evidence with source reference, producer/version and completeness scope. Generated evidence is authoritative only within supported generator scope; declarations can be wrong. Inferred evidence includes assumptions/confidence; observed evidence includes time window, environment and sampling. Raw SQL/custom code can make effect sets partial or unknown. Absence of an inferred/observed edge never proves absence of behavior or grants permission.

Observed facts are an operational overlay bound to a definition fingerprint, never mutations of the immutable AMG. Inspection exposes stale observations. Operation nodes declare inputs/outputs, authorization, transactions, cancellation, durability and effects; invokes/reads/writes/emits/requires edges link transports to shared business logic. See [Operations / Services](22-application-operations-and-services.md).

### Projection-specific fingerprints

Maintain deterministic application, database, api, auth, admin, jobs, realtime, docs and mcp fingerprints plus node hashes. Include projection schema/version, canonical semantic fields and relevant dependency closure. Database identity covers stored columns/constraints/persisted relationships; API covers wire schemas/routes/errors; auth covers policies/scopes; admin covers exposure/mutation modes; jobs covers envelopes/retry contracts; realtime covers protocols/policies; docs covers canonical content; MCP covers exposed contracts/capabilities.

Exclude absolute paths, timestamps, process IDs, addresses, runtime values, observations and unordered maps. An admin-label change cannot invalidate migrations; prose cannot invalidate wire clients. Policy changes invalidate affected admin/MCP/realtime contracts through dependency closure. Schema tools use database fingerprints, clients use API fingerprints, mutation preconditions use relevant operation/auth/MCP identities. Hashes detect change, not compatibility or authority. Algorithm/canonicalization remain VALIDATION REQUIRED.


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (H4, H10, M2, M5, M6, L4, L5, M1). Syntax is illustrative; claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment and retained sketches below.

### Registration of roots only

Applications register **roots**: models, operations/routes/resources, jobs, listeners, channels, settings, admin registrations and commands. Schemas, enums and nested types referenced by a root's typed signature are collected transitively through the generated metadata traits. Explicit `.schema::<T>()` registration (as in the "Registration Model" sketch below) is needed only for schemas that no root references. The validation error "Route references missing schema" therefore cannot occur for typed references. Unregistered roots produce no hidden behaviour. `rjango check` lists models, policies and jobs that are declared in source but not registered: the analogue of a forgotten `INSTALLED_APPS` entry.

### Where checks run

The AMG is built from compile-generated descriptors when the application starts, or in metadata-only introspection mode ([24](24-developer-experience-and-diagnostics.md)). Cross-type validation (relation targets, foreign-key compatibility, policy references, exposure declarations) is an **AMG validation error**, reported at startup and by `rjango check` with source spans. It is not a compile error. The "compile-generated, verified before the server starts" north star means *before serving*, not *at compile time*.

### Metadata identity lockfile

Stable IDs default to `app.name` derived from Rust names. Exposed IDs (models, fields, operations, routes, jobs, events, channels, MCP tools, error codes) are recorded in a checked-in `rjango.ids.lock`, maintained by `rjango check --update-ids` and reviewed like migrations. When a Rust rename would change a recorded ID, the change is reported as removed plus added and must be resolved: keep the old ID with `#[rjango(id = "...")]`, or accept the new ID and explicitly invalidate in-flight idempotency keys, previews and client contracts. Behaviour in the "Route Metadata" and "Edge Types" sketches below, where `route:users.create_user` and `route:users.create` are both used, is illustrative. Rename-stable field identity for migrations follows the same lockfile.

### Fingerprints detect declared change, not behaviour

Projection fingerprints hash declared metadata. An edit to a policy function body changes behaviour without changing the auth fingerprint. Policy descriptors therefore include a declared `version` and a macro-computed hash of the annotated policy function's token stream. That hash is not transitive across helper functions; this limitation is documented, and the Candidate 1 claim that policy changes invalidate dependent contracts holds only for declared metadata and the annotated body. MCP and preview preconditions must re-run authorization at execution time and never treat a matching fingerprint as behavioural equivalence ([MCP](../ai/mcp-architecture.md)).

### Type system corrections

- Unsigned scalars (`U16`/`U32`/`U64` in "Type System" below) are excluded from **DatabaseType** in 1.0. PostgreSQL has no unsigned integers. Model fields use signed types with `#[range(min = 0)]`, which emits a `CHECK` constraint, and the macro suggests this when an unsigned type is used. Unsigned types remain valid **WireType**s ([07](07-schemas-and-validation.md)).
- A model without a primary key is an error, not a warning (L4).
- "Public endpoint returns internal model directly" becomes a compile error: models never implement output schemas ([07](07-schemas-and-validation.md)), so the AMG warning is unnecessary.
- Paths in sketches below written `/users/:id` are SUPERSEDED by `/users/{id}` (Axum 0.8 syntax; [06](06-http-and-routing.md)).
- "Rjango v0.1 should support these node types" is historical scoping language; node coverage is defined by the subsystem specifications.
- MCP context-size figures below ("80,000" versus "3,000" tokens) are illustrative, not measured.

## Open decisions and interpretation

Exact storage representation, macro/public trait ergonomics, extension evolution, rename-stable identities, and complete feature-node coverage remain open. Graph examples are structural sketches.

## Scope and precedence

The original AMG design is retained in detail below. Later subsystem specifications expand the metadata requirements for admin, jobs, events, channels, storage and configuration. Their runtime state, business records and secret values still do not belong in the immutable definition graph.

The early implementation-order section (AMG §67) is SUPERSEDED by the design-first boundary and intentionally excluded. No execution sequence is created here.

<!-- Source: amg section 1. -->
## Purpose

The **Rjango Application Metadata Graph**, abbreviated **AMG**, is the canonical machine-readable representation of a Rjango application's structure.

It describes what the application **is**.

Examples:

- applications/modules
- models
- fields
- relationships
- constraints
- schemas
- routes
- handlers
- permissions
- authentication policies
- jobs
- commands
- migrations
- settings definitions
- services
- realtime channels

It does **not** contain application business data.

The graph should be consumable by:

```
                    Application Metadata Graph
                               │
         ┌─────────────────────┼─────────────────────┐
         │                     │                     │
      Admin UI              OpenAPI                 MCP
         │                     │                     │
         ▼                     ▼                     ▼
   CRUD interfaces        API schema           AI Agents

         │                     │                     │
         ├─────────────┬───────┴──────┬──────────────┤
         ▼             ▼              ▼              ▼
      CLI tools     Migrations    SDK Generator   Diagnostics
```

The AMG is therefore a foundational subsystem, not an AI feature.

---

<!-- Source: amg section 2. -->
## Core Principle

The graph is:

> **Generated from application code, validated by Rjango, and immutable while the application is running.**

The graph must never become a competing source of truth.

Wrong:

```
Source Code
    ↓

Metadata Graph ← AI modifies graph directly
    ↓

Application
```

Correct:

```
Human / AI
     ↓
Source Code
     ↓
Rjango Compilation / Registration
     ↓
Metadata Graph
     ↓
Framework Features
```

An AI tool may propose or generate source changes.

After compilation:

```
new source
   ↓
new graph
```

---

<!-- Source: amg section 3. -->
## Three Information Planes

We should deliberately separate three different kinds of information.

### Definition Plane

The canonical AMG.

Describes application structure.

Examples:

```
User model exists
User.email is String
User.email is unique
GET /users/:id exists
Endpoint returns UserResponse
Endpoint requires IsAuthenticated
```

This data is mostly deterministic.

---

### Runtime Plane

Describes the currently running instance.

Examples:

```
Environment: development
Database: connected
Database driver: PostgreSQL
Worker count: 12
Migrations pending: 0
Redis: connected
Application version: 0.4.2
```

This data changes.

It should **reference** AMG IDs but should not live inside the canonical graph.

---

### Operational Plane

Observability information.

Examples:

```
GET /users/:id
average latency: 8ms

query:model:User
12 calls

job:email.send
failed 2 times
```

Operational information references graph entities:

```
Trace
  route_id → route:users.detail

Slow query
  model_id → model:users.User
```

This gives us a powerful connection:

```
Application Definition
       +
Runtime State
       +
Observability
       =
Rjango understands the application
```

---

<!-- Source: amg section 4. -->
## Graph Structure

The graph contains:

```
pub struct ApplicationGraph {
    pub schema_version: MetadataVersion,
    pub application: ApplicationDescriptor,
    pub nodes: NodeRegistry,
    pub edges: EdgeRegistry,
    pub fingerprint: GraphFingerprint,
}
```

Conceptually:

```
Node ── Edge ── Node
```

Example:

```
App: users
     │
     └──contains──► Model: User
                         │
                         ├──contains──► Field: email
                         │
                         └──related_to──► Model: Organization
```

---

<!-- Source: amg section 5. -->
## Stable Metadata IDs

Every graph entity receives a stable human-readable ID.

Examples:

```
app:users

model:users.User

field:users.User.email

schema:users.CreateUser

route:users.create

permission:auth.IsAuthenticated

job:users.send_welcome_email

command:users.import_users
```

Rust type names alone should not be the identity.

Use:

```
pub struct MetadataId(String);
```

IDs should survive:

- process restarts
- recompilation
- deployment
- serialization
- MCP requests

They are the foreign keys of the Rjango ecosystem.

---

<!-- Source: amg section 6. -->
## Qualified Names

Objects also receive structured names.

```
pub struct QualifiedName {
    pub app: String,
    pub name: String,
}
```

Example:

```
app  = "users"
name = "User"
```

Canonical:

```
users.User
```

This allows:

```
User
```

to exist in multiple applications.

---

<!-- Source: amg section 7. -->
## Initial Core Node Types

Rjango v0.1 should support these node types.

```
pub enum NodeKind {
    Application,
    Model,
    Field,
    Relationship,
    Index,
    Constraint,

    Schema,
    SchemaField,

    Route,
    Handler,

    Permission,
    AuthPolicy,

    Job,
    Command,

    Migration,

    Setting,

    Service,
}
```

Later:

```
Event
Listener
Channel
WebSocket
AdminView
Cache
Storage
Agent
AgentTool
Workflow
Plugin
```

Do not put all future types into v0.1 prematurely.

---

<!-- Source: amg section 8. -->
## Application Nodes

Example:

```
pub struct ApplicationDescriptor {
    pub id: MetadataId,
    pub name: String,
    pub version: Option<String>,
    pub description: Option<String>,
}
```

Example:

```
app:users
```

owns:

```
model:users.User
schema:users.CreateUser
route:users.create
job:users.send_welcome_email
```

---

<!-- Source: amg section 9. -->
## Model Metadata

Example Rust:

```
#[derive(Model)]
pub struct User {
    #[primary_key]
    pub id: Uuid,

    #[unique]
    #[index]
    pub email: String,

    pub name: String,

    #[default = true]
    pub active: bool,

    #[auto_now_add]
    pub created_at: DateTime<Utc>,
}
```

The derive generates something conceptually equivalent to:

```
ModelDescriptor {
    id: "model:users.User",
    table: "users",
    fields: [...],
    indexes: [...],
    constraints: [...],
}
```

---

<!-- Source: amg section 10. -->
## Fields

Fields require normalized metadata.

```
pub struct FieldDescriptor {
    pub id: MetadataId,
    pub name: String,

    pub field_type: FieldType,

    pub nullable: bool,
    pub primary_key: bool,
    pub unique: bool,

    pub default: Option<DefaultDescriptor>,

    pub database: DatabaseFieldOptions,
}
```

---

<!-- Source: amg section 11. -->
## Type System

Do not expose raw Rust `TypeId` as our metadata contract.

We need a portable normalized type system.

Initial types:

```
pub enum ScalarType {
    Bool,

    I16,
    I32,
    I64,

    U16,
    U32,
    U64,

    F32,
    F64,

    String,
    Text,

    Uuid,

    Date,
    Time,
    DateTime,

    Decimal,

    Json,
    Bytes,
}
```

Containers:

```
pub enum FieldType {
    Scalar(ScalarType),

    Optional(Box<FieldType>),

    List(Box<FieldType>),

    Custom(CustomTypeDescriptor),
}
```

This normalized representation can later generate:

```
PostgreSQL
OpenAPI
JSON Schema
TypeScript
Swift
Kotlin
Admin inputs
```

---

<!-- Source: amg section 12. -->
## Relationship Metadata

Relationships should be explicit graph structures.

Examples:

```
User
   belongs_to
Organization
```

or:

```
Organization
   has_many
Users
```

Descriptor:

```
pub struct RelationshipDescriptor {
    pub id: MetadataId,

    pub source_model: MetadataId,
    pub target_model: MetadataId,

    pub kind: RelationshipKind,

    pub foreign_key: MetadataId,

    pub on_delete: ReferentialAction,
}
```

Kinds:

```
pub enum RelationshipKind {
    OneToOne,
    OneToMany,
    ManyToOne,
    ManyToMany,
}
```

---

<!-- Source: amg section 13. -->
## Constraints

Constraints should exist independently from fields.

```
pub enum ConstraintDescriptor {
    Unique(...),
    Check(...),
    ForeignKey(...),
}
```

This supports:

```
unique(email)

unique(organization_id, username)

check(price >= 0)
```

---

<!-- Source: amg section 14. -->
## Index Metadata

Indexes become graph nodes.

Example:

```
IndexDescriptor {
    id: "index:users.User.email",
    model: "model:users.User",
    fields: ["field:users.User.email"],
    unique: true,
}
```

This allows future Rjango diagnostics to detect:

```
frequently-filtered column without index
```

by combining:

```
AMG
+
query telemetry
```

---

<!-- Source: amg section 15. -->
## Schema Metadata

Database models and API schemas must remain separate.

Example:

```
#[derive(Schema)]
pub struct CreateUser {
    #[email]
    pub email: String,

    #[min_length(2)]
    pub name: String,
}
```

Descriptor:

```
schema:users.CreateUser
```

contains:

```
schema_field:users.CreateUser.email
schema_field:users.CreateUser.name
```

This schema can drive:

```
validation
OpenAPI
JSON Schema
TypeScript
MCP understanding
Admin forms
```

---

<!-- Source: amg section 16. -->
## Route Metadata

Example:

```
#[post("/users")]
#[permission(IsAuthenticated)]
async fn create_user(
    Json(input): Json<CreateUser>,
) -> Result<UserResponse> {
    ...
}
```

Produces:

```
RouteDescriptor {
    id: "route:users.create_user",

    method: POST,
    path: "/users",

    input: Some("schema:users.CreateUser"),
    output: Some("schema:users.UserResponse"),

    permissions: [
        "permission:auth.IsAuthenticated"
    ],
}
```

---

<!-- Source: amg section 17. -->
## Handler Metadata

Routes and handlers should technically be separate concepts.

Why?

One handler may eventually be attached to multiple routes.

```
Handler
   ▲
   │ handled_by
   │
Route
```

Handler metadata can include:

```
pub struct HandlerDescriptor {
    pub id: MetadataId,

    pub rust_path: String,

    pub async_handler: bool,

    pub source: Option<SourceReference>,
}
```

---

<!-- Source: amg section 18. -->
## Source References

For development builds, metadata should point back to source code.

```
pub struct SourceReference {
    pub file: String,
    pub line: u32,
    pub column: u32,
    pub module: String,
}
```

Example:

```
{
  "file": "src/users/routes.rs",
  "line": 42,
  "column": 1
}
```

This is extremely useful for:

```
CLI diagnostics
IDE integration
MCP agents
error reporting
```

Production metadata should be able to strip or redact filesystem paths.

---

<!-- Source: amg section 19. -->
## Permission Metadata

Example:

```
#[permission(IsAuthenticated)]
```

creates/reference:

```
permission:auth.IsAuthenticated
```

Descriptor:

```
pub struct PermissionDescriptor {
    pub id: MetadataId,
    pub name: String,
    pub description: Option<String>,
}
```

Routes then have:

```
route:users.update

    protected_by

permission:auth.IsAuthenticated
```

---

<!-- Source: amg section 20. -->
## Job Metadata

Example:

```
#[job]
async fn send_welcome_email(user_id: Uuid) -> Result<()> {
}
```

Descriptor:

```
JobDescriptor {
    id: "job:users.send_welcome_email",
    input_schema: ...,
    retry_policy: ...,
}
```

Future runtime data:

```
last execution
failure count
queue latency
```

belongs to the Runtime/Operational planes, not the AMG.

---

<!-- Source: amg section 21. -->
## Settings Metadata

Never store secret values.

Metadata should describe configuration:

```
SettingDescriptor {
    id: "setting:database.url",
    type: String,
    required: true,
    secret: true,
}
```

AMG can know:

```
DATABASE_URL exists as a setting definition.
```

It must never contain:

```
postgres://username:password@...
```

---

<!-- Source: amg section 22. -->
## Edge Types

The graph needs typed relationships.

Initial set:

```
pub enum EdgeKind {
    Contains,

    HasField,

    HasRelationship,

    UsesSchema,

    Accepts,

    Returns,

    Handles,

    ProtectedBy,

    BelongsTo,

    HasMany,

    HasOne,

    DependsOn,

    Reads,

    Writes,

    Migrates,
}
```

Example:

```
app:users
   └── contains ──► model:users.User

model:users.User
   └── has_field ──► field:users.User.email

route:users.create
   ├── accepts ──► schema:users.CreateUser
   ├── returns ──► schema:users.UserResponse
   └── protected_by ──► permission:auth.IsAuthenticated
```

---

<!-- Source: amg section 23. -->
## Typed Graph + Specialized Registries

Internally, we should provide both.

Generic graph:

```
graph.node(id)
graph.neighbors(id)
graph.edges_from(id)
```

Typed APIs:

```
graph.models()
graph.routes()
graph.schemas()
graph.jobs()
```

And:

```
graph.model("users.User")
```

This avoids forcing normal framework features to manually traverse graph edges.

---

<!-- Source: amg section 24. -->
## Registration Model

Avoid excessive runtime reflection.

Avoid a giant global mutable registry.

Recommended model:

```
Procedural Macro
       ↓
Metadata Descriptor
       ↓
Rjango App Module
       ↓
Application Builder
       ↓
Metadata Builder
       ↓
Validation
       ↓
Immutable ApplicationGraph
```

Example:

```
pub fn users_app() -> RjangoApp {
    RjangoApp::new("users")
        .model::<User>()
        .schema::<CreateUser>()
        .route(create_user)
}
```

Rjango macros generate the descriptors.

The application explicitly registers components.

This provides:

- deterministic startup
- understandable ownership
- easier testing
- plugin isolation
- fewer linker tricks
- less global magic

---

<!-- Source: amg section 25. -->
## Macro Design

Example:

```
#[derive(Model)]
pub struct User {
    ...
}
```

could generate:

```
impl RjangoModel for User {
    fn metadata() -> ModelDescriptor {
        ...
    }
}
```

Likewise:

```
#[derive(Schema)]
```

generates:

```
impl RjangoSchema for CreateUser {
    fn metadata() -> SchemaDescriptor {
        ...
    }
}
```

Route macros generate:

```
RouteDescriptor
```

alongside the Axum-compatible handler registration.

---

<!-- Source: amg section 26. -->
## Build Pipeline

Metadata construction should have explicit stages.

```
1. COLLECT
      ↓
2. NORMALIZE
      ↓
3. RESOLVE
      ↓
4. VALIDATE
      ↓
5. FINGERPRINT
      ↓
6. FREEZE
```

### Collect

Gather descriptors from registered Rjango applications.

### Normalize

Convert metadata into canonical forms.

### Resolve

Resolve references:

```
ForeignKey<User>
→ model:users.User
```

### Validate

Detect problems.

### Fingerprint

Calculate deterministic hashes.

### Freeze

Produce immutable `ApplicationGraph`.

---

<!-- Source: amg section 27. -->
## Validation

The graph should reject structurally invalid applications before serving requests.

Examples:

```
Unknown relationship target

Duplicate model ID

Duplicate route

Route references missing schema

Permission reference missing

Foreign key points to incompatible type

Duplicate database table

Invalid index definition
```

Example:

```
RJG-META-021

Model `orders.Order` references unknown model
`customers.Customer`.

src/orders/models.rs:31

    pub customer: ForeignKey<Customer>
                  ^^^^^^^^^^^^^^^^^^^^^
```

Framework quality here matters enormously.

---

<!-- Source: amg section 28. -->
## Warnings

> **Candidate 2 note:** "Model has no primary key" below is an AMG error, and "Public endpoint returns internal model directly" is a compile error. Neither is a warning.

Not everything should stop compilation/startup.

Warnings:

```
Model has no primary key

Public endpoint returns internal model directly

Route has no description

Frequently used foreign key has no index

MCP tool exposes write capability in development

Deprecated metadata feature
```

These become part of:

```
rjango check
```

---

<!-- Source: amg section 29. -->
## Deterministic Fingerprints

Every node receives a structural fingerprint.

Example:

```
model:users.User

fingerprint:
5b72c7...
```

Fingerprints should exclude irrelevant values such as:

```
memory address
build timestamp
absolute source path
runtime state
```

Include:

```
field structure
types
relationships
constraints
configuration that affects semantics
```

The entire graph also gets:

```
graph_fingerprint
```

---

<!-- Source: amg section 30. -->
## Why Fingerprints Matter

They enable:

#### Migration detection

```
previous model graph
       vs
current model graph
```

#### Client generation

Don't regenerate TypeScript if API graph hasn't changed.

#### Development tooling

Detect structural reload.

#### AI tools

Agent can verify:

```
I analyzed graph fingerprint X.
Current graph is Y.
My context is stale.
```

This could be extremely valuable.

---

<!-- Source: amg section 31. -->
## Metadata Versioning

AMG schema version and Rjango framework version are different concepts.

Example:

```
{
  "metadata_schema": "1.0",
  "rjango_version": "0.7.2"
}
```

Rules:

```
metadata major version change
→ potentially breaking

metadata minor version change
→ backward compatible additions
```

This lets external tools depend on the metadata protocol without tightly coupling themselves to one Rjango build.

---

<!-- Source: amg section 32. -->
## Serialized Format

CLI:

```
rjango inspect --json
```

Example:

```
{
  "metadata_schema": "1.0",
  "application": {
    "name": "acme"
  },
  "nodes": [
    {
      "id": "model:users.User",
      "kind": "model",
      "name": "User"
    },
    {
      "id": "field:users.User.email",
      "kind": "field",
      "name": "email",
      "type": {
        "scalar": "string"
      }
    }
  ],
  "edges": [
    {
      "from": "model:users.User",
      "to": "field:users.User.email",
      "kind": "has_field"
    }
  ]
}
```

---

<!-- Source: amg section 33. -->
## Canonical Serialization

Canonical JSON output should be deterministic.

Same application:

```
same node ordering
same edge ordering
same normalized values
```

produces:

```
same fingerprint
```

This matters for:

- CI
- code generation
- migrations
- caching
- tests
- AI context

---

<!-- Source: amg section 34. -->
## Query API

Rust:

```
let metadata = app.metadata();

let user = metadata
    .models()
    .get("users.User")?;
```

Routes using model:

```
metadata.routes_for_model("users.User")
```

Permissions:

```
metadata.permissions_for_route("users.update")
```

Relationships:

```
metadata.relationships("users.User")
```

---

<!-- Source: amg section 35. -->
## Generic Graph Querying

Advanced API:

```
metadata
    .query()
    .from("model:users.User")
    .edge(EdgeKind::HasRelationship)
    .execute();
```

But normal framework components should use typed interfaces.

---

<!-- Source: amg section 36. -->
## CLI Introspection

Commands:

```
rjango inspect
```

Summary:

```
Application: acme

Apps:          4
Models:       12
Fields:       94
Schemas:      18
Routes:       34
Permissions:   7
Jobs:          5
```

Specific model:

```
rjango inspect model users.User
```

Result:

```
Model: users.User
Table: users

Fields:

id          UUID        PRIMARY KEY
email       String      UNIQUE INDEX
name        String
active      Bool        default=true
created_at  DateTime

Relationships:

organization → organizations.Organization

Used by routes:

GET   /users/:id
PATCH /users/:id
```

---

<!-- Source: amg section 37. -->
## Graph Exploration

Future:

```
rjango inspect deps users.User
```

Could show:

```
users.User
 ├── organization → organizations.Organization
 ├── returned_by → GET /users/:id
 ├── returned_by → GET /users
 └── used_by → job:users.send_welcome_email
```

---

<!-- Source: amg section 38. -->
## Migration Integration

This needs a strict architecture.

The AMG represents:

> **desired application structure**

It is NOT automatically the current database structure.

Migration engine compares:

```
Desired Model Graph

vs

Known Migration State
```

and optionally:

```
Database Introspection
```

Pipeline:

```
Current migration state
        │
        ▼
Previous schema representation

        VS

Current AMG model representation
        │
        ▼
Schema Difference
        │
        ▼
Migration Proposal
```

---

<!-- Source: amg section 39. -->
## Migration Operations

Graph differences become explicit operations.

Examples:

```
MigrationOperation::CreateTable(...)

MigrationOperation::AddColumn(...)

MigrationOperation::DropColumn(...)

MigrationOperation::AlterColumn(...)

MigrationOperation::CreateIndex(...)

MigrationOperation::AddForeignKey(...)
```

AI may help generate or explain migrations.

It should not be required for migration correctness.

---

<!-- Source: amg section 40. -->
## Migration Safety

Rjango should classify migration operations.

Example:

```
SAFE

+ nullable column
+ index

REVIEW

~ column type change

DESTRUCTIVE

- table
- column
```

Example:

```
Migration 0042

✓ Add `nickname` nullable text

⚠ DROP COLUMN `legacy_username`

This migration contains destructive operations.
```

Eventually AI can explain risks, but classification should be deterministic.

---

<!-- Source: amg section 41. -->
## OpenAPI Integration

OpenAPI becomes an AMG projection.

```
AMG
 │
 ├── Route metadata
 ├── Schema metadata
 ├── Validation metadata
 ├── Authentication metadata
 │
 ▼
OpenAPI
```

There should be no separate set of API annotations required merely to produce documentation.

---

<!-- Source: amg section 42. -->
## TypeScript Generation

Likewise:

```
AMG
 │
 ├── Schemas
 ├── Routes
 ├── Enums
 └── Validation constraints
 │
 ▼
TypeScript SDK
```

Example:

```
#[derive(Schema)]
pub struct CreateVenue {
    pub name: String,
}
```

generates:

```
export interface CreateVenue {
    name: string;
}
```

Routes generate:

```
client.venues.create(data)
```

---

<!-- Source: amg section 43. -->
## Admin Integration

Admin uses metadata:

```
Model
Fields
Relationships
Validation
Permissions
```

plus optional admin-specific configuration.

Therefore:

```
AMG core metadata
        +
Admin metadata
        ↓
Admin Interface
```

Admin does not maintain its own model definition system.

---

<!-- Source: amg section 44. -->
## `rjango check` Integration

Many checks become graph queries.

Example:

```
for each public route:
    ensure response schema exists
```

or:

```
for each relationship:
    ensure target model exists
```

or:

```
for each unique field:
    ensure database constraint exists
```

The AMG becomes the foundation of Rjango diagnostics.

---

<!-- Source: amg section 45. -->
## MCP Integration

MCP resources can directly project AMG data.

Examples:

```
rjango://application

rjango://models

rjango://models/users.User

rjango://routes

rjango://routes/users.create

rjango://permissions
```

Agent asks:

```
inspect_model("users.User")
```

Rjango responds from the graph.

No repository search is necessary for basic architectural understanding.

---

<!-- Source: amg section 46. -->
## MCP Responses Should Reference Metadata IDs

Example:

```
{
  "id": "model:users.User",
  "fields": [
    {
      "id": "field:users.User.email",
      "type": "string",
      "unique": true
    }
  ]
}
```

Then agent requests:

```
find_routes_using("model:users.User")
```

instead of passing ambiguous names around.

---

<!-- Source: amg section 47. -->
## MCP Context Efficiency

This is a huge advantage.

Instead of sending an agent:

```
80,000 tokens of Rust source
```

Rjango could provide:

```
3,000 tokens of precise structural metadata
```

Then source is fetched only when needed.

That should make agents:

- faster
- cheaper
- less confused
- more accurate

---

<!-- Source: amg section 48. -->
## AI Change Workflow

Suppose the developer says:

> Add a description to Venue.

Agent:

```
inspect_model("venues.Venue")
```

receives:

```
model location
fields
routes
schemas
relationships
fingerprint
```

Agent proposes source patch.

After edit:

```
rjango check
```

Rjango rebuilds graph.

Agent compares:

```
old fingerprint
new fingerprint
```

Then:

```
generate migration
run tests
regenerate client
```

This is much safer than letting an agent modify database state independently.

---

<!-- Source: amg section 49. -->
## Metadata Exposure Policies

Not all metadata should be exposed everywhere.

Every node can receive an exposure classification.

Example:

```
pub enum MetadataExposure {
    Public,
    Internal,
    DevelopmentOnly,
    Restricted,
}
```

Example:

```
Route structure        Internal
OpenAPI routes         Public
Source filesystem path DevelopmentOnly
Secret setting         Restricted
```

---

<!-- Source: amg section 50. -->
## Sensitive Metadata

AMG should NEVER include secret values.

Examples that must not appear:

```
database password
API key
JWT signing secret
OAuth secret
private key
session secret
```

It may include:

```
setting exists
setting type
setting is required
setting is secret
```

---

<!-- Source: amg section 51. -->
## MCP Authorization

Metadata query:

```
inspect_model
```

might be low risk.

But:

```
read_database_rows
```

is not part of the AMG itself and requires separate runtime authorization.

This separation is important.

```
AMG
     read-only application structure

Runtime Tools
     potentially privileged operations
```

---

<!-- Source: amg section 52. -->
## Extension Metadata

Plugins need a safe extension mechanism.

Do not allow core enums to become enormous.

Each node may contain:

```
pub struct ExtensionMetadata {
    pub namespace: String,
    pub version: String,
    pub data: serde_json::Value,
}
```

Example:

```
namespace:
rjango.stripe
```

or:

```
rjango.redis
```

Plugins own their extension schema.

---

<!-- Source: amg section 53. -->
## Extension Namespace Rules

Reserved:

```
rjango.*
```

Third-party examples:

```
acme.audit
some_crate.analytics
```

Namespaces prevent collisions.

---

<!-- Source: amg section 54. -->
## Provenance

Metadata should record where it came from.

```
pub enum MetadataOrigin {
    Core,
    Application,
    Plugin {
        name: String,
        version: String,
    },
    Generated,
}
```

Useful when diagnosing:

```
Why does this route exist?
```

Rjango can answer:

```
Registered by plugin `rjango-auth`
```

---

<!-- Source: amg section 55. -->
## Immutability

After startup graph validation:

```
Arc<ApplicationGraph>
```

becomes read-only.

Requests can cheaply share it across threads.

No locks are needed for graph mutation because there is no mutation.

This fits perfectly with Rjango's concurrent execution architecture.

---

<!-- Source: amg section 56. -->
## Performance

Metadata should be constructed once.

```
startup
   ↓
build graph
   ↓
freeze
   ↓
share Arc<ApplicationGraph>
```

Request processing should not rebuild descriptors.

Indexes should support O(1) common lookup:

```
MetadataId → Node
QualifiedName → Model
RouteId → Route
```

Adjacency lists handle graph traversal.

---

<!-- Source: amg section 57. -->
## Internal Storage

Potential structure:

```
pub struct ApplicationGraph {
    nodes: HashMap<MetadataId, Node>,
    outgoing: HashMap<MetadataId, Vec<Edge>>,
    incoming: HashMap<MetadataId, Vec<Edge>>,

    models: ModelRegistry,
    schemas: SchemaRegistry,
    routes: RouteRegistry,

    fingerprint: GraphFingerprint,
}
```

We get both:

```
fast normal framework operations
```

and:

```
flexible graph traversal
```

---

<!-- Source: amg section 58. -->
## Development Hot Reload

Longer term:

```
old graph
   ↓
recompile
   ↓
new graph
   ↓
GraphDiff
```

Example:

```
Metadata change detected

+ field:venues.Venue.description

Affected systems:

Migration       required
OpenAPI         changed
TypeScript SDK  changed
Admin           changed
```

That developer experience could be exceptional.

---

<!-- Source: amg section 59. -->
## Graph Diff

Define:

```
pub struct GraphDiff {
    pub added: Vec<MetadataId>,
    pub removed: Vec<MetadataId>,
    pub changed: Vec<NodeChange>,
}
```

Example:

```
Added
 + field:venues.Venue.description

Changed
 ~ schema:venues.VenueResponse

Affected
 ~ route:venues.detail
 ~ route:venues.list
```

This can power an enormous amount of tooling.

---

<!-- Source: amg section 60. -->
## Dependency Impact Analysis

Eventually:

```
rjango impact model venues.Venue
```

could report:

```
Changing venues.Venue may affect:

Database
  venues table

API
  GET /venues
  GET /venues/:id
  POST /venues

Schemas
  VenueResponse
  CreateVenue

Clients
  TypeScript Venue type

Admin
  VenueAdmin

Jobs
  reindex_venue
```

This would be useful to both humans and coding agents.

---

<!-- Source: amg section 61. -->
## Public HTTP Exposure

Do NOT automatically expose:

```
/__rjango/metadata
```

in production.

Development could optionally expose a protected endpoint.

Preferred mechanisms:

```
CLI
MCP
library API
```

HTTP metadata inspection should be explicitly enabled.

---

<!-- Source: amg section 62. -->
## Rjango Metadata API

Rust:

```
let graph = app.metadata();
```

Examples:

```
graph.model("users.User")?;

graph.route("users.create")?;

graph.routes_using_schema("users.CreateUser");

graph.models_referencing("organizations.Organization");
```

---

<!-- Source: amg section 63. -->
## Initial Crate Location

I would put the core graph implementation in:

```
rjango-core
```

with specialized descriptors owned by their respective crates.

Example:

```
rjango-core
   MetadataId
   Node
   Edge
   ApplicationGraph
   MetadataBuilder
   MetadataVersion

rjango-models
   ModelDescriptor
   FieldDescriptor
   RelationshipDescriptor

rjango-web
   RouteDescriptor
   HandlerDescriptor

rjango-auth
   PermissionDescriptor

rjango-jobs
   JobDescriptor
```

This prevents `rjango-core` from knowing every future framework feature.

---

<!-- Source: amg section 64. -->
## Trait-Based Metadata

Core contract:

```
pub trait MetadataProvider {
    fn metadata(&self) -> MetadataFragment;
}
```

More specialized:

```
pub trait RjangoModel {
    fn model_metadata() -> ModelDescriptor;
}
```

```
pub trait RjangoSchema {
    fn schema_metadata() -> SchemaDescriptor;
}
```

Macros implement these automatically.

---

<!-- Source: amg section 65. -->
## MetadataFragment

Each subsystem contributes fragments.

```
pub struct MetadataFragment {
    pub nodes: Vec<Node>,
    pub edges: Vec<Edge>,
}
```

Application builder:

```
builder.register(fragment);
```

Finally:

```
let graph = builder.build()?;
```

---

<!-- Source: amg section 66. -->
## Important Architectural Rule

**Feature crates contribute metadata. They do not own the application graph.**

Wrong:

```
rjango-models has registry
rjango-web has separate registry
rjango-auth has another registry
```

Correct:

```
rjango-models ─┐
rjango-web ────┤
rjango-auth ───┼──► MetadataBuilder ─► ApplicationGraph
rjango-jobs ───┤
plugins ───────┘
```

This is critical.

---

<!-- Source: amg section 68. -->
## First Prototype

> **Candidate 2 note:** paths are written `/users/{id}`, and handlers return derived output schemas. See the amendment above.

Our first end-to-end AMG test should contain:

```
#[derive(Model)]
struct User {
    #[primary_key]
    id: Uuid,

    #[unique]
    email: String,
}

#[derive(Schema)]
struct UserResponse {
    id: Uuid,
    email: String,
}

#[get("/users/:id")]
async fn get_user(...) -> Result<UserResponse> {
    ...
}
```

Then:

```
rjango inspect
```

should produce a graph containing:

```
app:test

model:test.User
 ├── field:test.User.id
 └── field:test.User.email

schema:test.UserResponse
 ├── schema_field:id
 └── schema_field:email

route:test.get_user
 ├── GET /users/:id
 ├── returns → test.UserResponse
 └── handled_by → test.get_user
```

---

<!-- Source: amg section 69. -->
## First MCP Prototype

An agent asks:

```
inspect_model("test.User")
```

Response:

```
{
  "id": "model:test.User",
  "table": "users",
  "fields": [
    {
      "name": "id",
      "type": "uuid",
      "primary_key": true
    },
    {
      "name": "email",
      "type": "string",
      "unique": true
    }
  ]
}
```

Then:

```
find_routes_using("test.User")
```

Rjango can derive connected routes through graph traversal.

---

<!-- Source: amg section 70. -->
## Long-Term Potential

Once this exists, Rjango can eventually answer questions most frameworks cannot answer directly.

Examples:

```
What changes if I remove this model?

Which APIs expose this field?

Which routes can mutate User?

Which models contain personally identifying fields?

Which endpoints depend on this permission?

Which frontend types are invalidated by this change?

Which migration introduced this field?

Which jobs touch Venue?

Which recent errors came from code associated with this route?
```

The framework itself has enough structural knowledge to answer.

---

<!-- Source: amg section 71. -->
## Architectural North Star

A Rjango application should increasingly behave like a **self-describing program**.

Not reflection in the loose dynamic-language sense.

Instead:

> **Explicit, typed, compile-generated, deterministic structural knowledge.**

Rust gives us an unusually good foundation for this because much of the metadata can be verified before the server starts.

The resulting stack becomes:

```
                    SOURCE CODE
                         │
                         ▼
                 Compile-Time Metadata
                         │
                         ▼
               Application Metadata Graph
                         │
          ┌──────────────┼───────────────┐
          ▼              ▼               ▼
       Runtime        Developer          AI
       Features         Tools          Agents
```

And that graph becomes the architectural spine of Rjango.
