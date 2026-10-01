# Caching

[Master specification](README.md) · [Status and provenance](preservation.md)

**Decision status:** SPEC-LOCKED for the stated architecture and invariants; PROPOSED for examples, alternatives, and explicitly open choices.
**Evidence status:** VALIDATION REQUIRED.
**Source:** Part IV.

Caching has explicit namespaces, serialization versions, TTLs, invalidation, and tenant/security boundaries. The cache must not silently become authoritative storage.

> All commands, Rust types, generated output, tests, and performance results shown as examples are design illustrations. This documentation does not establish that Rjango implements them or that they have passed validation.

## Review Candidate 2 amendment

**Status:** architectural direction adopted from the [independent review](../reviews/independent-review-candidate-1.md) (H8, M11). This topic had no Candidate 1 amendment; this section reconciles it with [tenancy](10-authorization-and-security.md). Syntax is illustrative; claims remain VALIDATION REQUIRED.

- **Typed keys.** Keys are typed (illustrative: `#[rjango::cache_key("venue_summary", version = 2)] struct VenueSummaryKey { venue_id: Uuid }`). The framework prefixes the application, environment, key name, schema version and **the tenant from `Ctx`** automatically. Raw string keys (`cache.set("venue:123", ...)` below) are SUPERSEDED for application data. Tenant-independent data requires an explicit `global` key declaration, which `rjango check` lists.
- **Authorization-dependent values.** Cached values derived from policy-scoped queries include the policy identity and the actor's scope-relevant attributes in the key, or are cached only after scoping at a tenant-wide grain. Cache-aside helpers take the scoped `Db` and refuse `SystemDb` results unless the key is declared `global`.
- **Serialization.** Values serialize with the key's declared version. A version mismatch is a miss, never a guess.
- **Hidden I/O.** Cache calls are explicit `.await`ed operations. Per-request caches are in-memory only.
- **No secrets.** `Secret<T>` does not implement serialization, so it cannot be cached by construction ([18](18-configuration.md)).

## Open decisions and interpretation

See the retained lock/open list. Backend defaults and distributed stampede/invalidation behavior remain open.



<!-- Source: iv section 45. -->
## Caching

### Goal

Rjango caching should be explicit enough to understand but convenient enough to use routinely.

No invisible global caching that makes correctness impossible to reason about.

---

<!-- Source: iv section 46. -->
## Cache Backends

Common trait:

```
CacheBackend
```

Potential implementations:

```
memory
Redis
custom distributed cache
```

---

<!-- Source: iv section 47. -->
## Cache Namespaces

Keys should automatically include application/framework namespace information.

Conceptually:

```
rjango:
  app:
    environment:
      cache_version:
        user_key
```

This avoids accidental cross-environment/key collisions.

---

<!-- Source: iv section 48. -->
## Typed Cache API

> **SUPERSEDED (Candidate 2):** raw string keys are replaced by typed, tenant-prefixed keys; see the amendment above.

Conceptual:

```
cache
    .set("venue:123", &venue, ttl)
    .await?;
```

Serialization should be type-aware.

But arbitrary Rust type serialization must have a defined stability contract.

---

<!-- Source: iv section 49. -->
## Cache Serialization Versioning

Cached structures evolve.

Rjango should support a schema/version component in cache keys or values.

Changing:

```
VenueSummary v1
→ VenueSummary v2
```

must not silently deserialize old incompatible bytes.

---

<!-- Source: iv section 50. -->
## TTL

Explicit time-to-live encouraged.

Avoid permanent cache entries unless deliberately configured.

---

<!-- Source: iv section 51. -->
## Cache-Aside Helpers

Common pattern:

```
read cache
   ↓ miss
read DB
   ↓
populate cache
```

Rjango can provide helpers without hiding behavior.

---

<!-- Source: iv section 52. -->
## Stampede Protection

When 10,000 requests miss the same expensive key:

```
one computes
others wait/use stale
```

rather than all recomputing.

Support:

```
single-flight
stale-while-revalidate
```

where appropriate.

---

<!-- Source: iv section 53. -->
## Negative Caching

Optional caching of:

```
not found
```

must have shorter/explicit TTLs to avoid hiding newly created resources.

---

<!-- Source: iv section 54. -->
## Cache Invalidation

Rjango should resist promising automatic perfect invalidation.

Explicit invalidation APIs:

```
delete key
delete namespace/tag
version bump
```

Tag-based invalidation may be useful.

---

<!-- Source: iv section 55. -->
## Query Caching

Do not automatically cache arbitrary ORM queries.

Potential explicit syntax:

```
.cache_for(...)
```

can be explored later.

Default:

```
database query
→ database query
```

Predictability first.

---

<!-- Source: iv section 56. -->
## Per-Request Cache

A request-local cache can safely deduplicate identical work within one request.

Useful for:

```
permission lookups
data loader behavior
repeated service calls
```

This is distinct from distributed caching.

---

<!-- Source: iv section 57. -->
## Sensitive Data in Cache

Caching sensitive data should require awareness of:

```
backend encryption
shared cache environment
TTL
tenant separation
```

Secret values should not accidentally be placed into generalized caches.

---

<!-- Source: iv section 58. -->
## Cache Observability

Metrics:

```
hit
miss
set
delete
latency
errors
stampede wait
```

Rjango can expose cache behavior in traces.

---

<!-- Source: iv section 133. -->
## Testing — Cache

Required:

```
hit/miss
expiry
serialization mismatch
stampede
distributed contention
backend outage
negative caching
namespace collision
tenant isolation
```

---

<!-- Source: iv section 142. -->
## Current Lock Status — Cache

#### LOCK

- pluggable backend
- explicit behavior
- namespacing
- TTL support
- instrumentation
- stampede protection capability

#### OPEN

- default production backend
- automatic query-caching APIs
- tag invalidation implementation
