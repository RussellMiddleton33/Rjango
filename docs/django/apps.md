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
