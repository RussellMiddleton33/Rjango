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
