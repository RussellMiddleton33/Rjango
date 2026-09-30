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
