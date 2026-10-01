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

> **Candidate 2:** the transaction is an owning guard (`let mut tx = db.begin().await?; ... tx.commit().await?;`), or the convenience `db.atomic(async |tx| ...)`. **Where the analogy stops:** Django's `atomic()` is ambient, so every ORM call inside it joins the transaction. In Rjango only queries passed `&mut tx` do. Using the outer `db` inside the block runs on another connection outside the transaction and can wait on the transaction's own locks; Rjango warns with `RJG-DB-TX-OUTSIDE`. `commit()` consumes `tx` and can report an unknown outcome. Use `tx.dispatch`/`tx.emit` where Django uses `on_commit`. See [ADR 0013](../adr/0013-transaction-ownership.md).

## Reading loaded relations (Candidate 2)

`.with(Venue::R.floors)` yields `Loaded<Venue>` items. Read loaded data with `venue.floors()?`, which returns `Result<&[Loaded<Floor>], NotLoaded>`. Forgetting `.with` gives a `NotLoaded` error naming the call to add, never a hidden query. Django instead queries lazily on attribute access. See [ADR 0014](../adr/0014-loaded-relations.md).
