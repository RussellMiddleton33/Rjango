# Web, identity and security: coming from Django

[Guide index](README.md) · [Canonical specification](../specification/10-authorization-and-security.md)

**Status:** PROPOSED guide, grounded in the agreed design. **Evidence:** VALIDATION REQUIRED. Examples are illustrative; commands and APIs are not asserted to exist. This scaffold retains the original comparison and must gain version-tested examples with implementation.

<!-- Source: iii section 111. -->
## Django Transition Documentation

These systems all receive side-by-side Django explanations.

Examples:

```
Django URLconf
→ Rjango typed routes

Django middleware
→ Tower/Rjango middleware

DRF Serializer
→ Rjango Schema

DRF ViewSet
→ Rjango Resource

AuthenticationBackend
→ Rjango authenticator

Permission classes
→ Rjango permission policies

CSRF middleware
→ Rjango CSRF policy

INSTALLED_APPS
→ explicit Rjango app registration
```

Every page must explain:

> where the analogy stops.

## Candidate 2 differences

- **Deny by default.** Every handler, resource, admin registration, channel and MCP tool declares a policy or `public`. Django views are open unless decorated.
- **Permission classes become operation policies** with `scope` (list filtering) and `check` (object) forms. They receive `ctx`, not `request`.
- **`DRF Serializer` becomes a derived schema** (`#[rjango::schema(from = Model, output, fields(...))]`) with explicit, compile-checked field lists.
- **`DRF ViewSet` becomes a Resource** generating scoped operations.
- **WebSockets.** Cookie-authenticated WebSocket/SSE handshakes require a trusted `Origin`. Django Channels leaves this to `AllowedHostsOriginValidator`; Rjango enforces it.
- **Settings.** An unset environment means production. Security-sensitive settings in the base `rjango.toml` apply only to development and test.
- **Passwords.** Django password hashes can be imported and are rehashed on login.

See [Everyday workflow](everyday-workflow.md).
