# Applications and modules

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III.

Applications are explicitly registered Rust modules or crates with stable identities, declared dependencies, reusable namespaces, typed configuration, and an ordered lifecycle.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Open decisions and interpretation

The full plugin/package compatibility contract remains unspecced; app descriptor and lifecycle signatures are illustrative.



<!-- Source: iii section 20. -->
## Rjango Applications / Modules

Rjango should preserve Django's excellent idea of dividing a project into cohesive reusable applications.

But the terminology should remain Rust-friendly.

Potential name:

```
Rjango App
```

is clear and familiar.

Example structure:

```
src/
├── users/
│   ├── mod.rs
│   ├── models.rs
│   ├── routes.rs
│   ├── schemas.rs
│   ├── services.rs
│   ├── jobs.rs
│   └── admin.rs
│
├── venues/
│   └── ...
```

---

<!-- Source: iii section 21. -->
## App Definition

Potential:

```
pub fn app() -> RjangoApp {
    RjangoApp::new("venues")
        .models(models::metadata())
        .routes(routes())
        .jobs(jobs())
        .admin(admin())
}
```

Application root:

```
Rjango::new()
    .app(users::app())
    .app(venues::app())
```

Explicit registration remains preferred over filesystem discovery magic.

---

<!-- Source: iii section 22. -->
## App Identity

Each application has a stable identifier:

```
app:venues
```

and metadata:

```
AppDescriptor {
    name,
    version,
    description,
    dependencies,
    source,
}
```

App labels become part of migration and metadata identity.

Changing them later should be treated as potentially breaking.

---

<!-- Source: iii section 23. -->
## App Dependencies

Example:

```
orders
  depends_on
users

venues
  depends_on
organizations
```

Dependencies should be explicit.

This lets Rjango determine:

- startup order
- migration order
- plugin compatibility
- missing dependencies
- circular dependencies

Cycles should generally be rejected unless a specific subsystem explicitly supports them.

---

<!-- Source: iii section 24. -->
## App Lifecycle

Possible lifecycle:

```
register
   ↓
configure
   ↓
validate
   ↓
startup
   ↓
ready
   ↓
shutdown
```

Apps may participate through explicit lifecycle hooks.

Examples:

```
async fn startup(...)
async fn shutdown(...)
```

But normal apps should rarely need them.

---

<!-- Source: iii section 25. -->
## Reusable Apps

A reusable Rjango application should be publishable as a normal Rust crate.

Example:

```
rjango-audit
rjango-comments
rjango-stripe
```

Application:

```
[dependencies]
rjango-audit = "1"
```

then:

```
Rjango::new()
    .app(rjango_audit::app())
```

---

<!-- Source: iii section 26. -->
## Reusable App Isolation

Reusable apps should namespace:

- metadata IDs
- routes
- migrations
- settings
- commands
- admin elements
- MCP extensions

This prevents collisions.

---

<!-- Source: iii section 27. -->
## App Configuration

Reusable apps should expose typed configuration.

Example:

```
AuditConfig {
    retention_days: 365,
}
```

Registration:

```
.app(
    rjango_audit::app()
        .configure(AuditConfig {
            retention_days: 365,
        })
)
```

Avoid string-key configuration when Rust types can prevent mistakes.

---

<!-- Source: iii section 28. -->
## Application Metadata

Apps become first-class AMG nodes.

```
app:venues
 ├── Model
 ├── Schema
 ├── Route
 ├── Permission
 ├── Job
 ├── Command
 └── Migration
```

This makes:

```
"What belongs to this app?"
```

trivial for humans and agents to answer.

---

<!-- Source: iii section 29. -->
## Django Mapping — Apps

```
Django AppConfig            Rjango AppDescriptor

INSTALLED_APPS              explicit .app(...)

ready()                     lifecycle startup/ready

app label                   stable app ID

reusable Django package     reusable Rust crate
```

Main improvement:

> Registration is explicit and type-safe instead of driven primarily by import strings.
