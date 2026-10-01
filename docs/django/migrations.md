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

## Candidate 2 differences

- **Artifacts.** Migrations are a reviewable data IR. `RunPython`-style data steps are Rust functions against a stable `rjango-migrate` API, using historical rows (`ctx.table(...)`) like Django's `apps.get_model`.
- **Deploy tags.** Each operation is tagged expand, contract or breaking for rolling deploys. Plans tell you which steps run before and after deploying, which Django leaves to convention.
- **Readiness.** Production instances never auto-apply. An instance whose binary is outside the schema compatibility window stays alive but not ready.
- **Development auto-apply.** `rjango dev` auto-applies only against a database created with `--dev`.
- **Renames.** These need `#[renamed_from]`, or an expand/contract sequence with `--rolling`.
