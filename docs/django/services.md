# Application services: coming from Django

[Guide index](README.md) · [Canonical specification](../specification/20-cross-system-invariants.md)

**Status:** PROPOSED guide, grounded in the agreed design. **Evidence:** VALIDATION REQUIRED. Examples are illustrative; commands and APIs are not asserted to exist. This scaffold retains the original comparison and must gain version-tested examples with implementation.

<!-- Source: iv section 126. -->
## Django Transition — Jobs

Documentation should explicitly say:

```
Django itself
does not provide a durable distributed job queue.

Typical Django:
Celery/RQ/Huey/etc.

Rjango:
standardized job abstraction built into the framework contract.
```

This prevents overstating Django parity.

---

<!-- Source: iv section 127. -->
## Django Transition — Caching

```
Django cache framework
→ Rjango CacheBackend

cache.get/set
→ similar concept

per-view caching
→ explicit Rjango HTTP/cache integration

cached template fragments
→ relevant only to future template system
```

---

<!-- Source: iv section 128. -->
## Django Transition — Storage

```
Django Storage
→ Rjango StorageBackend

FileField/ImageField
→ Rjango stored-file model integration

MEDIA_ROOT
→ local storage configuration

S3 backend
→ storage provider
```

---

<!-- Source: iv section 129. -->
## Django Transition — Email

```
django.core.mail
→ Rjango email service

console email backend
→ Rjango dev backend

email backend
→ EmailBackend
```

Major improvement:

> Production queuing is intentionally designed alongside the jobs system.

---

<!-- Source: iv section 130. -->
## Django Transition — Signals

Important guide:

```
Django signal
→ sometimes Rjango lifecycle hook
→ sometimes Rjango domain event
→ sometimes Rjango job
```

Documentation should help developers choose based on semantics rather than offer a single mechanical replacement.

Example:

```
post_save to normalize one field
→ model hook

post_save to send email
→ domain event + job

post_save to notify websocket users
→ domain event → realtime broadcast
```

This could be one of the most useful Django migration guides.
