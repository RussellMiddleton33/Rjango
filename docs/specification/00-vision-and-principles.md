# Vision and principles

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Initial master specification.

Rjango combines Django-style productivity with Rust correctness, async execution, explicit dependencies, and structured application metadata. Its application must operate fully without an AI model.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Open decisions and interpretation

The final 1.0 scope and release contract remain open. Early release milestones in historical discussion are superseded by the requirement to finish the design first.



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
