# Coming from Django

[Master specification](../specification/README.md)

**Status:** SPEC-LOCKED as an adoption/documentation requirement; PROPOSED guide structure. **Evidence:** VALIDATION REQUIRED.

Each guide explains the familiar Django concept, the Rjango design counterpart, and where the analogy stops. These pages describe a framework design, not a released migration tool or working API. Rjango examples require an explicit database context; earlier `User::objects()` examples in the discussion are superseded.

| Topic | Rjango design | Guide |
| --- | --- | --- |
| Models | Typed model declarations and generated metadata; final macro form open | [Models](models.md) |
| Managers and QuerySets | Lazy typed queries; explicit async terminals and DB context | [Queries](queries.md) |
| Migrations | Reviewable operations, history DAG and explicit production application | [Migrations](migrations.md) |
| Apps | Registered modules/crates with stable identities and dependencies | [Applications](apps.md) |
| Admin | Explicitly registered AMG projection with permissions and audit | [Admin](admin.md) |
| Jobs, caching, storage, email, signals | Integrated service contracts with explicit effects and durability | [Application services](services.md) |
| Views, middleware, serializers, authentication, settings | Initial conceptual mappings preserved | [Web and security](web-and-security.md) |
| Everyday workflow (views, `request.user`, `atomic`, `on_commit`, shell, runserver) | Tier 0 operations, scoped data, owning transactions | [Everyday workflow](everyday-workflow.md) |
| Learning Rust through Rjango | Ordered concept curriculum with typical errors | [Rust concepts](rust-concepts.md) |

**Candidate 2:** start with [Everyday workflow](everyday-workflow.md). Where older guide pages conflict with it (for example the ambient-looking `atomic` closure mapping), the Candidate 2 direction governs.

## Differences to teach explicitly

Async execution and Rust ownership replace many dynamic Python assumptions. Use `Result` and `Option`, explicit pool/transaction context, typed registration and compile-generated metadata. Relationship access cannot secretly query the database. Background task spawning does not imply durability. Configuration is typed and validated. Django deployments may already use threads, processes or async; the comparison must not portray Django as inherently single-threaded.

## Remaining guide design

The complete curriculum, tested examples, version support and concept coverage tooling remain unspecced. CLI translation, templates/forms/static assets, internationalization, deployment, and detailed authentication/middleware migration guides require fuller designs. A future version-aware MCP translation resource or prompts may explain equivalents, convert examples, and identify limitations; these are proposed capabilities, not available commands.
