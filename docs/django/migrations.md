# Migrations: coming from Django

[Guide index](README.md) · [Canonical specification](../specification/04-migrations.md)

**Status:** PROPOSED guide, grounded in the agreed design. **Evidence:** VALIDATION REQUIRED. Examples are illustrative; commands and APIs are not asserted to exist. This scaffold retains the original comparison and must gain version-tested examples with implementation.

<!-- Source: iii section 18. -->
## Django Mapping — Migrations

```
Django                    Rjango

makemigrations            rjango migration make

migrate                   rjango migration run

showmigrations             rjango migration list

squashmigrations           rjango migration squash

RunPython                  data migration

RunSQL                     raw SQL operation

migration dependencies     migration DAG
```

Rjango adds first-class AMG fingerprints, drift analysis, structured safety classification and agent-facing introspection.
