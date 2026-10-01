# Applications and modules

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part III.

Applications are explicitly registered Rust modules or crates with stable identities, declared dependencies, reusable namespaces, typed configuration, and an ordered lifecycle.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 1 amendment

**Status:** architectural requirements adopted; syntax and experimental claims remain VALIDATION REQUIRED. This amendment takes precedence over conflicting historical sketches below.

### Plugin trust and macro/runtime compatibility

1.0 plugins are **compile-time trusted Rust extensions** linked into the application with process authority. Namespaces/capabilities do not sandbox Rust code. No stable dynamic Rust ABI or untrusted hot-loaded binary isolation is promised. Untrusted extensions need a separate process/protocol outside this contract. Declare dependencies, feature requirements, extension schema versions and supported Rjango versions; unknown required extensions fail registration.

Proc macros and runtime share a versioned generated-code contract. Incompatible pairs/duplicate runtimes fail with actionable diagnostics before serving traffic. Exact pairing versus supported version ranges, MSRV and features remain PROPOSED. Generated internal symbols are not public application APIs. Compile-pass/fail tests cover renamed dependencies, reexports, feature combinations, workspace skew and upgrades.


## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (M4, M8, H10). Syntax is illustrative; claims remain VALIDATION REQUIRED. Takes precedence over the Candidate 1 amendment where they conflict.

### Reusable apps reference host identities by contract, not by type

A reusable crate cannot name the host application's Rust types. Direction:

- **Auth user contract.** The project registers exactly one auth user model (illustrative: `.auth_user::<User>()`). Its primary key type is the framework type `rjango::auth::UserId` (a UUID newtype). Reusable apps store `UserId` fields and declare a foreign key to "the configured auth user model". The AMG resolves this at the Resolve stage and migration generation emits the concrete foreign key in the host's migration graph. This replaces Django's swappable `AUTH_USER_MODEL` with a typed contract and no generics.
- **Other host references.** These use the same pattern: a framework-defined ID type plus a named contract, such as tenant (`TenantId`) or a contract the reusable app declares that the host binds at registration. Generic app parameters (`rjango_comments::app::<MyUser>()`) are not the default because they spread type parameters into user code and error messages.
- **Migrations.** Reusable-app migrations depend on contract nodes ("auth user table exists"), not on host migration IDs. `rjango check` fails if a contract is unbound.

### Configuration of reusable apps

Typed configuration structs are the schema. Values come from the layered configuration ([18](18-configuration.md)) under the app's namespace, and `.configure(...)` in code supplies defaults that configuration may override per environment. Secrets are never accepted as code literals.

### Version skew

The claim that incompatible macro/runtime pairs fail "before serving traffic" is corrected. Duplicate runtimes are made a **dependency-resolution error**: lockstep exact versions plus a Cargo `links` key. Macros resolve the facade's actual crate name, so renamed dependencies work, and reach internals only through its hidden `__private` module. See [24](24-developer-experience-and-diagnostics.md).

### Registration

App builders register roots only; referenced schemas are collected transitively ([02](02-application-metadata-graph.md)). The `.models(models::metadata())` style below is illustrative.

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
