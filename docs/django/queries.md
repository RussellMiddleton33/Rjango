# Queries and transactions: coming from Django

[Guide index](README.md) · [Canonical specification](../specification/03-models-and-orm.md)

**Status:** PROPOSED guide, grounded in the agreed design. **Evidence:** VALIDATION REQUIRED. Examples are illustrative; commands and APIs are not asserted to exist. This scaffold retains the original comparison and must gain version-tested examples with implementation.

<!-- Source: orm section 54. -->
## Django `select_related`

Documentation:

```
Django:
select_related()

Rjango:
.with(...)
```

Rjango chooses/loading strategy based on relation structure.

---

<!-- Source: orm section 55. -->
## Django `prefetch_related`

Same Rjango abstraction:

```
.with(Venue::R.floors)
```

We should avoid forcing Django's two APIs onto Rjango if the underlying loader can choose the appropriate strategy itself.

This is an example of:

> Preserve the concept, improve the interface.

---

<!-- Source: orm section 56. -->
## Django `save()`

Django:

```
venue.name = "New Name"
venue.save()
```

Rjango:

```
venue
    .edit()
    .name("New Name")
    .save(&db)
    .await?;
```

Difference should be explicitly documented:

Rjango uses an explicit change-tracking edit operation rather than silently observing arbitrary object mutation.

---

<!-- Source: orm section 57. -->
## Django `atomic`

Django:

```
with transaction.atomic():
    ...
```

Rjango:

```
db.transaction(|tx| async move {
    ...
}).await?;
```
