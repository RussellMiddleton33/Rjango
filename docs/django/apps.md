# Applications: coming from Django

[Guide index](README.md) · [Canonical specification](../specification/05-app-system.md)

**Status:** PROPOSED guide, grounded in the agreed design. **Evidence:** VALIDATION REQUIRED. Examples are illustrative; commands and APIs are not asserted to exist. This scaffold retains the original comparison and must gain version-tested examples with implementation.

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

## Candidate 2 differences

- **Roots only.** Register roots (models, handlers/operations, resources, jobs, admin); schemas are collected from their signatures. `rjango check` lists declared but unregistered models, the counterpart of a missing `INSTALLED_APPS` entry.
- **Swappable user.** `AUTH_USER_MODEL` becomes `.auth_user::<User>()`, with `rjango::auth::UserId` as its primary key type. Reusable apps reference users and tenants through ID contracts, not host types.
- **Stable IDs.** Exposed IDs are recorded in a checked-in `rjango.ids.lock`. Renaming a Rust function that backs an exposed operation is reported, like a migration.
