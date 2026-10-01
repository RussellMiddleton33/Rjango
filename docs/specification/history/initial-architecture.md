# Initial architecture — historical design source

[Master specification](../README.md) · [Preservation record](../preservation.md)

**Status:** PROPOSED historical material; SUPERSEDED wherever later topic specifications refine it. **Evidence:** VALIDATION REQUIRED.

> **Candidate 2 reading note:** examples here using `/users/:id` paths, handlers returning models, `User::objects()`, `#[permission(...)]` as business authorization, `send_welcome_email::dispatch(...)` outside a transaction, a base `rjango.toml` with `environment = "development"` and `[mcp] enabled = true`, and the `secrets.read` MCP permission are all SUPERSEDED. See the [master specification](../README.md) precedence rules and [20](../20-cross-system-invariants.md) for the conforming programming model.

This retains the initial architecture to avoid losing early rationale and outlines for areas not yet fully specced. Read the canonical topic documents first. Early examples using ambient database lookup, automatic CRUD exposure, mutable metadata, or unqualified MCP mutation permission are superseded by explicit DB context, opt-in exposure, immutable AMG, and separate permissions. API syntax, CLI commands, generated output and performance claims are illustrative.

The original sections 35–38 described development phases, initial milestones, engineering tickets, and an early v0.1 scope. They are SUPERSEDED by the later instruction to finish all design and the final cross-system review before any implementation planning. They are intentionally not reproduced as an active or historical execution plan. Early “Phase one/Later” labels elsewhere are historical scope suggestions, not current sequencing. The final 1.0 release contract remains open.

<!-- Source: master section 1. -->
## Vision

Rjango is an async-first, type-safe, AI-native web application framework for Rust.

It should provide the productivity and conventions that make Django attractive while embracing the strengths of modern Rust:

- Compile-time safety
- Async I/O
- Multi-core execution
- High concurrency
- Predictable performance
- Strong typing
- Explicit security boundaries
- Modern API development
- Real-time communication
- Generated frontend clients
- Machine-readable application metadata
- Native MCP integration
- AI-agent-friendly development

Rjango is **AI-native but never AI-dependent**.

A Rjango application must work completely without an AI model.

AI agents gain additional abilities because Rjango intentionally exposes structured information about the application through safe, permissioned interfaces.

The framework's north-star experience is:

```
cargo install rjango

rjango new myapp
cd myapp

rjango dev
```

Then:

```
Rjango 0.1

✓ Application loaded
✓ PostgreSQL connected
✓ 7 migrations applied
✓ 12 routes registered
✓ 4 models registered
✓ Development server running
✓ MCP development server available

http://localhost:8000
```

---

<!-- Source: master section 2. -->
## Core Philosophy

Rjango should follow ten architectural principles.

### Convention over configuration

The common case should require almost no configuration.

Developers should be able to create models, APIs, authentication and admin interfaces without manually assembling dozens of libraries.

Advanced developers must still have escape hatches.

---

### Async-first

Async is not an optional mode.

Rjango's normal execution model is:

```
HTTP Request
     ↓
Async Task
     ↓
Tokio Runtime
     ↓
Worker Threads
```

One request does **not** equal one operating-system thread.

Thousands of lightweight async tasks can execute across a relatively small number of runtime threads.

---

### Multi-core by default

The default Rjango runtime uses Tokio's multi-thread scheduler.

Applications should naturally use available CPU cores without developers configuring worker processes manually.

---

### Safe shared state

Framework-managed application state must satisfy Rust's concurrency guarantees.

Rjango internally manages concepts such as:

```
Arc<AppState>
```

without forcing application developers to deal with `Arc`, locks or runtime internals for normal application development.

---

### Batteries included, internals replaceable

Rjango should provide an excellent default stack.

Initial stack:

```
Runtime           Tokio
HTTP              Axum
Middleware        Tower / tower-http
Database          PostgreSQL
ORM               Rjango Models → SeaORM
SQL engine        SQLx / SeaQuery
Serialization     Serde
Observability     tracing
OpenAPI           Generated from Rjango metadata
CLI               Rjango CLI
AI integration    MCP
```

But Rjango should not unnecessarily prevent direct use of Axum, Tower, SQLx or lower-level Rust libraries.

---

### Metadata is infrastructure

This is one of Rjango's most important architectural principles.

Everything that Rjango understands about an application should be representable in a structured **Application Metadata Graph**.

For example:

```
Application
 ├── Models
 │    ├── Fields
 │    ├── Relationships
 │    ├── Constraints
 │    └── Indexes
 │
 ├── Routes
 │    ├── Parameters
 │    ├── Responses
 │    └── Permissions
 │
 ├── Jobs
 ├── Commands
 ├── Authentication
 ├── Permissions
 ├── Migrations
 ├── Settings
 └── Services
```

This metadata becomes the common source for:

```
             Application Metadata
                     │
     ┌───────────────┼─────────────────┐
     │               │                 │
   Admin           OpenAPI            MCP
     │               │                 │
   CLI        TypeScript SDK       AI Agents
     │               │                 │
   Docs            Testing          Diagnostics
```

We should avoid independently implementing six different forms of application introspection.

---

### API-first, not API-only

Rjango should be optimized initially for applications where the frontend may be:

- SvelteKit
- React
- Vue
- Mobile applications
- Desktop applications
- Other services

Server rendering can exist, but it is not the architectural center of Rjango.

---

### AI-native

AI agents should be able to understand Rjango applications through structured APIs rather than blindly searching source code.

Examples:

```
inspect_models
inspect_model
inspect_routes
inspect_migrations
inspect_permissions
inspect_errors
inspect_logs
run_tests
run_checks
create_migration
generate_client
```

---

### Secure-by-default

Dangerous capabilities must require explicit authorization.

Development convenience must never silently become production exposure.

---

### Excellent errors are a framework feature

Rust compiler errors can become intimidating when macros and traits are involved.

Rjango must invest heavily in:

- actionable compile errors
- runtime diagnostics
- `rjango check`
- migration diagnostics
- configuration validation
- human-readable errors
- machine-readable errors for agents

---

<!-- Source: master section 3. -->
## High-Level Architecture

```
                    Rjango Application
                           │
          ┌────────────────┼────────────────┐
          │                │                │
        HTTP             Jobs             MCP
          │                │                │
          ▼                ▼                ▼
      Rjango Web      Rjango Tasks    Rjango Intelligence
          │                │                │
          └────────────────┼────────────────┘
                           │
                  Application Metadata
                           │
          ┌────────────────┼────────────────┐
          │                │                │
        Models          Services         Security
          │
      Rjango Data
          │
       SeaORM
          │
      SQLx / SQL
          │
      PostgreSQL

-------------------------------------------------

Framework runtime:

Tokio → Axum → Tower → Rjango
```

Rjango should not implement its own HTTP server or async runtime.

Our innovation happens **above those primitives**.

---

<!-- Source: master section 4. -->
## Runtime Architecture

### Request model

Each incoming request becomes an asynchronous task.

```
Request A ─┐
Request B ─┼── Tokio scheduler ── CPU cores
Request C ─┤
Request D ─┘
```

While Request A waits for PostgreSQL:

```
A → waiting

CPU executes B/C/D

database returns

A → resumes
```

This allows extremely high I/O concurrency without requiring one OS thread per connection.

---

<!-- Source: master section 5. -->
## Blocking Work

Blocking or CPU-heavy operations must not accidentally stall Tokio runtime threads.

Rjango should expose:

```
rjango::task::blocking(|| {
    perform_cpu_heavy_operation()
}).await?;
```

Potential future macro:

```
#[rjango::blocking]
fn create_thumbnail(...) {
}
```

Documentation must clearly distinguish:

#### Async workloads

```
Database
Redis
HTTP APIs
S3
Network operations
Async filesystem APIs
```

#### Blocking/CPU workloads

```
Image processing
Compression
Large PDF generation
Video operations
CPU-heavy calculations
Synchronous legacy libraries
```

---

<!-- Source: master section 6. -->
## Workspace Architecture

Rjango should be a Cargo workspace rather than one enormous crate.

Proposed structure:

```
rjango/
├── crates/
│   ├── rjango/
│   ├── rjango-core/
│   ├── rjango-runtime/
│   ├── rjango-web/
│   ├── rjango-macros/
│   ├── rjango-config/
│   ├── rjango-models/
│   ├── rjango-db/
│   ├── rjango-migrations/
│   ├── rjango-auth/
│   ├── rjango-admin/
│   ├── rjango-jobs/
│   ├── rjango-openapi/
│   ├── rjango-client/
│   ├── rjango-mcp/
│   ├── rjango-testing/
│   └── rjango-cli/
│
├── examples/
├── docs/
├── benchmarks/
├── tests/
└── Cargo.toml
```

Users should normally depend only on:

```
[dependencies]
rjango = "..."
```

The umbrella crate re-exports the normal public API.

---

<!-- Source: master section 7. -->
## Rjango Application Structure

A generated project should initially resemble:

```
myapp/
├── Cargo.toml
├── rjango.toml
├── .env
│
├── src/
│   ├── main.rs
│   ├── app.rs
│   ├── settings.rs
│   │
│   ├── users/
│   │   ├── mod.rs
│   │   ├── models.rs
│   │   ├── routes.rs
│   │   ├── schemas.rs
│   │   ├── services.rs
│   │   └── admin.rs
│   │
│   └── venues/
│       └── ...
│
├── migrations/
└── tests/
```

We should retain the useful Django concept of **applications/modules** without requiring Python's exact project/app architecture.

---

<!-- Source: master section 8. -->
## Application Boot

Desired:

```
use rjango::prelude::*;

#[rjango::main]
async fn main() -> Result<()> {
    Rjango::new()
        .app(users::app())
        .app(venues::app())
        .run()
        .await
}
```

Potential generated version may be even simpler:

```
#[rjango::main]
async fn main() {
    app().run().await;
}
```

Rjango should handle:

- configuration
- logging
- database initialization
- connection pools
- metadata registration
- routes
- graceful shutdown
- framework checks
- lifecycle hooks

---

<!-- Source: master section 9. -->
## Models

Rjango's model syntax should feel immediately understandable to Django developers while remaining Rust-native.

Target experience:

```
#[derive(Model)]
#[rjango(table = "users")]
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

    #[auto_now]
    pub updated_at: DateTime<Utc>,
}
```

Relationships:

```
#[derive(Model)]
pub struct Venue {
    #[primary_key]
    pub id: Uuid,

    pub name: String,

    #[has_many]
    pub floors: HasMany<Floor>,
}
```

And:

```
#[derive(Model)]
pub struct Floor {
    #[primary_key]
    pub id: Uuid,

    #[belongs_to]
    pub venue: ForeignKey<Venue>,

    pub name: String,
}
```

---

<!-- Source: master section 10. -->
## Query API

Desired developer experience:

```
let users = User::objects()
    .filter(User::active.eq(true))
    .order_by(User::created_at.desc())
    .all()
    .await?;
```

Single object:

```
let user = User::objects()
    .get(id)
    .await?;
```

Creation:

```
let user = User::create()
    .email("russ@example.com")
    .name("Russ")
    .save()
    .await?;
```

Updates:

```
user
    .set_name("Russ Middleton")
    .save()
    .await?;
```

Transactions:

```
db.transaction(|tx| async move {
    let user = User::create(...).using(tx).await?;
    Organization::create(...).using(tx).await?;

    Ok(())
}).await?;
```

Rjango must also provide an escape hatch for direct SQL/SQLx.

---

<!-- Source: master section 11. -->
## Migrations

Target commands:

```
rjango migration make
rjango migration show
rjango migration plan
rjango migration run
rjango migration rollback
```

Django-like workflow:

```
Model changes
     ↓
Metadata comparison
     ↓
Migration generated
     ↓
Developer reviews
     ↓
Migration applied
```

Production migrations should never be silently generated or applied by AI.

---

<!-- Source: master section 12. -->
## Routing

Target:

```
#[get("/users/:id")]
async fn user_detail(
    Path(id): Path<Uuid>,
) -> Result<User> {
    User::objects().get(id).await
}
```

Creation:

```
#[post("/users")]
async fn create_user(
    Json(input): Json<CreateUser>,
) -> Result<User> {
    User::create(input).save().await
}
```

Routes should automatically contribute metadata describing:

- HTTP method
- path
- input schema
- output schema
- authentication
- permissions
- documentation
- source location

---

<!-- Source: master section 13. -->
## Automatic API Generation

Rjango should support an optional higher-level API abstraction.

Example:

```
#[rjango::api]
#[model(User)]
#[permissions(IsAuthenticated)]
pub struct UserApi;
```

Potential generated endpoints:

```
GET     /api/users
POST    /api/users
GET     /api/users/:id
PATCH   /api/users/:id
DELETE  /api/users/:id
```

Customization must remain straightforward.

---

<!-- Source: master section 14. -->
## Validation and Schemas

Input and database models must not necessarily be identical.

Example:

```
#[derive(Schema)]
pub struct CreateUser {
    #[email]
    pub email: String,

    #[min_length(2)]
    #[max_length(100)]
    pub name: String,
}
```

Validation errors should generate consistent machine-readable responses.

---

<!-- Source: master section 15. -->
## Error Model

Application handlers should return:

```
Result<T>
```

Framework errors should automatically map to suitable HTTP responses.

Examples:

```
NotFound         → 404
ValidationError  → 422
Unauthorized     → 401
Forbidden        → 403
Conflict         → 409
RateLimited      → 429
Internal         → 500
```

Development responses should contain rich diagnostics.

Production responses must not expose:

- stack traces
- secrets
- SQL credentials
- internal paths
- private implementation details

---

<!-- Source: master section 16. -->
## Authentication

Initial authentication architecture should support:

#### Phase one

- Password authentication
- Sessions
- Secure cookies
- JWT/API tokens
- Password reset
- Email verification

#### Later

- OAuth
- OIDC
- Google
- Microsoft
- GitHub
- Passkeys/WebAuthn
- Enterprise SSO
- Service accounts
- API keys

Authentication and authorization must remain separate concepts.

---

<!-- Source: master section 17. -->
## Permissions

Target:

```
#[get("/admin/reports")]
#[permission(IsAdmin)]
async fn reports() -> Result<Report> {
    ...
}
```

Composable permissions:

```
#[permission(IsAuthenticated + CanViewVenue)]
```

Object-level permissions should eventually be supported.

---

<!-- Source: master section 18. -->
## OpenAPI

OpenAPI should be generated automatically from Rjango metadata.

No separate manually-maintained API schema.

```
rjango openapi generate
```

Potential development endpoint:

```
/openapi.json
```

Interactive API documentation may be optionally provided.

---

<!-- Source: master section 19. -->
## Frontend Client Generation

One of Rjango's major modern features.

```
rjango client generate typescript
```

Given:

```
pub struct Venue {
    pub id: Uuid,
    pub name: String,
}
```

Generate:

```
export interface Venue {
    id: string;
    name: string;
}
```

And API methods:

```
venues.list()
venues.get(id)
venues.create(data)
venues.update(id, data)
venues.delete(id)
```

Longer term:

```
rjango client generate swift
rjango client generate kotlin
rjango client generate dart
```

TypeScript is first priority.

---

<!-- Source: master section 20. -->
## Admin

A Django-quality admin experience should eventually become one of Rjango's defining features.

Models can opt in:

```
#[derive(Admin)]
pub struct User {
    ...
}
```

Or:

```
#[rjango::admin(User)]
pub struct UserAdmin {
    #[list]
    fields: [email, name, created_at];

    #[search]
    fields: [email, name];

    #[filter]
    fields: [active, created_at];
}
```

Admin should provide:

- CRUD
- searching
- filtering
- sorting
- pagination
- relationship editing
- permissions
- audit history
- bulk actions

The admin interface itself can eventually consume Rjango metadata dynamically.

---

<!-- Source: master section 21. -->
## Background Jobs

Target:

```
#[job]
async fn send_welcome_email(user_id: Uuid) -> Result<()> {
    ...
}
```

Dispatch:

```
send_welcome_email::dispatch(user.id).await?;
```

Development may use an embedded executor.

Production:

```
rjango worker
```

Future capabilities:

- retries
- backoff
- delayed execution
- scheduled jobs
- priorities
- dead-letter queues
- concurrency controls
- observability

---

<!-- Source: master section 22. -->
## Real-Time

Rjango should eventually provide first-class:

- WebSockets
- Server-Sent Events
- Pub/Sub
- Channels
- presence
- broadcast groups

Potential abstraction:

```
#[channel("venue/:id")]
async fn venue_channel(
    socket: Socket,
    venue_id: Uuid,
) {
    ...
}
```

Redis should become the standard distributed transport where necessary.

---

<!-- Source: master section 23. -->
## The Rjango CLI

The CLI is a core product, not an afterthought.

```
rjango new
rjango dev
rjango run
rjango build

rjango app create

rjango model create
rjango model inspect

rjango migration make
rjango migration run
rjango migration rollback

rjango admin create-user

rjango routes
rjango shell
rjango test
rjango check

rjango openapi generate
rjango client generate

rjango worker

rjango inspect
rjango mcp
```

---

<!-- Source: master section 24. -->
## `rjango check`

`rjango check` should become one of our signature capabilities.

Example:

```
Rjango System Check

Runtime
✓ Tokio runtime configuration valid
✓ Application metadata loaded

Database
✓ PostgreSQL reachable
✓ Connection pool healthy
✓ All migrations applied

Routing
✓ 27 routes registered
✓ No duplicate route definitions

Models
✓ 8 models registered
✓ All relationships valid

Security
✓ Secure cookies enabled
✓ CSRF protection enabled
✓ Production debug disabled

API
✓ OpenAPI schema valid

AI
✓ MCP configuration valid
✓ Production mutation tools disabled

Warnings

⚠ Venue.slug is queried frequently but has no index
⚠ Possible N+1 query detected in VenueApi.list
```

Checks should have structured output:

```
rjango check --json
```

so AI agents and CI can consume the same diagnostic system.

---

<!-- Source: master section 25. -->
## AI Architecture

Rjango should not sprinkle AI calls into random framework components.

Instead:

```
                     AI Model
                        │
                    MCP Client
                        │
                 Rjango MCP Server
                        │
             Permission / Policy Layer
                        │
              Rjango Metadata + Tools
```

The framework remains deterministic.

---

<!-- Source: master section 26. -->
## Native MCP

Rjango should provide:

```
rjango mcp
```

Every Rjango application can become an MCP server in development.

The MCP interface should expose three categories.

### Resources

Read-only application knowledge.

```
rjango://application
rjango://models
rjango://models/User
rjango://routes
rjango://migrations
rjango://settings/schema
rjango://jobs
rjango://permissions
rjango://errors/recent
```

### Tools

Actions.

```
inspect_model
inspect_route
inspect_schema

run_check
run_test

generate_migration
validate_migration

query_database
generate_openapi
generate_typescript_client
```

Later:

```
create_model
create_route
create_job
```

### Prompts

Framework-guided workflows.

Examples:

```
Create Rjango model
Debug Rjango endpoint
Design database migration
Add authenticated API
Investigate failing request
```

---

<!-- Source: master section 27. -->
## MCP Security

This is mandatory architecture.

Example:

```
[mcp]
enabled = true

[mcp.permissions]

metadata.read = true

database.schema.read = true
database.data.read = false
database.data.write = false

migrations.generate = true
migrations.apply = false

tests.run = true

shell.execute = false

secrets.read = false
```

Production default:

```
[mcp]
enabled = false
```

Every MCP action should eventually have:

- identity
- authorization
- audit trail
- structured inputs
- structured output
- timeout
- scope
- environment restrictions

AI agents should receive capabilities, not unrestricted shell access.

---

<!-- Source: master section 28. -->
## Agent-Friendly Architecture

A coding agent should be able to ask:

```
What models exist?
```

and receive structured metadata rather than grep output.

Then:

```
Add organization support.
Users belong to organizations.
Organizations have owner/admin/member roles.
```

Possible agent workflow:

```
inspect_model(User)
        ↓
inspect_schema()
        ↓
inspect_routes()
        ↓
propose_changes()
        ↓
generate_migration()
        ↓
run_check()
        ↓
run_tests()
```

This is a major Rjango differentiator.

---

<!-- Source: master section 29. -->
## Observability

Tracing should be native.

Every request should receive:

- request ID
- span
- duration
- status
- route metadata

Optional integrations later:

- OpenTelemetry
- Sentry
- Prometheus
- distributed tracing

Agents could then safely query diagnostics:

```
get_recent_errors
inspect_request
inspect_slow_queries
```

---

<!-- Source: master section 30. -->
## Configuration

Default configuration:

```
[app]
name = "myapp"
environment = "development"

[server]
host = "127.0.0.1"
port = 8000

[database]
url_env = "DATABASE_URL"

[mcp]
enabled = true
```

Priority:

```
defaults
   ↓
rjango.toml
   ↓
environment configuration
   ↓
environment variables
```

Secrets should never be committed to configuration files by default.

---

<!-- Source: master section 31. -->
## Testing

Testing must feel much easier than typical Rust application testing.

Target:

```
#[rjango::test]
async fn user_can_login(app: TestApp) {
    let user = app.factory::<User>().create().await;

    let response = app
        .post("/login")
        .json(json!({
            "email": user.email,
            "password": "password"
        }))
        .send()
        .await;

    response.assert_ok();
}
```

Testing utilities should include:

- application harness
- isolated database
- factories
- fixtures
- authenticated clients
- request helpers
- response assertions
- migration testing

---

<!-- Source: master section 32. -->
## Framework Security

Security defaults should include:

- secure cookie settings
- CSRF protection where applicable
- CORS policy
- request-size limits
- timeout middleware
- secret redaction
- safe error responses
- SQL parameterization
- password hashing
- rate limiting
- trusted proxy configuration
- security headers

Rjango should warn when production configurations are unsafe.

---

<!-- Source: master section 33. -->
## Performance Philosophy

Rjango should prioritize:

- developer productivity
- correctness
- predictable performance
- raw benchmark performance

We should not destroy usability to win microbenchmarks.

However, framework abstraction should avoid unnecessary:

- allocations
- cloning
- locks
- serialization
- dynamic dispatch
- runtime reflection

Compile-time generation should be preferred where practical.

---

<!-- Source: master section 34. -->
## Escape Hatches

Rjango must never feel like a cage.

Developers should be able to reach:

```
Axum Router
Tower Service
SQLx connection
SeaORM entities
Tokio runtime
raw HTTP request
raw SQL
```

without abandoning the framework.

---

<!-- Source: master section 39. -->
## v1.0 Definition

Rjango reaches 1.0 when a developer can realistically choose it instead of Django/FastAPI/Rails/Laravel for a production backend.

That means:

```
Runtime
Routing
ORM
Migrations
Validation
Authentication
Authorization
OpenAPI
TypeScript clients
Admin
Jobs
Caching
Storage
Email
Testing
CLI
Observability
Security
MCP
Documentation
Stable public API
```

---

<!-- Source: master section 40. -->
## What Rjango Is Not

Rjango is not:

- another HTTP router
- another async runtime
- another Tokio replacement
- another SQL engine
- an AI wrapper
- an LLM framework pretending to be a web framework
- a direct line-by-line Rust port of Django

Instead:

> Rjango composes proven Rust infrastructure into a cohesive application framework and adds the conventions, metadata, tooling and AI interfaces required for dramatically higher developer productivity.

---

<!-- Source: master section 41. -->
## Product Identity

The core product promise:

> **Build modern applications at Django speed with Rust confidence.**

Possible shorter positioning:

> **Django thinking. Rust execution.**

AI-specific positioning:

> **The Rust web framework designed for humans and agents.**

---

<!-- Source: master section 42. -->
## Architectural North Star

Eventually, this should be possible:

```
rjango new acme
cd acme
rjango dev
```

A developer tells their coding agent:

> Add organizations. Users belong to organizations. Organizations have owner, admin and member roles. Create authenticated APIs, migrations, admin pages and TypeScript types.

The agent interacts with Rjango:

```
inspect_schema
inspect_model(User)
inspect_routes
        ↓
generate_changes
        ↓
create_migration
        ↓
run_check
        ↓
run_tests
```

Rjango reports:

```
✓ Organization model added
✓ User.organization relationship added
✓ Membership model added
✓ Role constraints created
✓ Migration generated
✓ 8 API routes registered
✓ Admin metadata registered
✓ TypeScript client regenerated
✓ OpenAPI schema updated
✓ 42 tests passing
```

The important innovation is not that AI wrote Rust.

The innovation is that **the framework understands its own application well enough to give both humans and AI safe, structured tools for changing it.**

That principle should influence every major architectural decision we make from this point forward.
