# Models: coming from Django

[Guide index](README.md) · [Canonical specification](../specification/03-models-and-orm.md)

**Status:** PROPOSED guide, grounded in the agreed design. **Evidence:** VALIDATION REQUIRED. Examples are illustrative; commands and APIs are not asserted to exist. This scaffold retains the original comparison and must gain version-tested examples with implementation.

<!-- Source: orm section 53. -->
## Django Mapping

Every ORM documentation page should contain a Django mapping where applicable.

Example:

#### Django

```
Venue.objects.filter(
    active=True
).order_by("-created_at")
```

#### Rjango

```
Venue::objects(&db)
    .filter(Venue::F.active.eq(true))
    .order_by(Venue::F.created_at.desc())
    .all()
    .await?;
```

Difference:

```
Django QuerySet
    lazy and database context mostly implicit

Rjango QuerySet
    lazy, typed, async and database context explicit
```

## Candidate 2 differences

- `db` is the **scoped** handle from `ctx.db()`. Tenant-owned models (`#[rjango::model(tenant = organization_id)]`) are filtered automatically. Django managers have no such default.
- Relations are declared on foreign-key fields. Rows contain no relation fields, and loaded relations are read via `Loaded<M>` accessors.
- A model generates `Venue`, `NewVenue`, `VenuePatch`, `Venue::F` and `Venue::R`, all visible in rustdoc.
- Models are never returned from API handlers. Derive output schemas with explicit field lists, the Rjango counterpart of `ModelSerializer`.
- `#[version]` gives optimistic concurrency, used by Admin and resource updates.
- Unsigned integer fields are not database types. Use signed types with `#[range(min = 0)]`.
